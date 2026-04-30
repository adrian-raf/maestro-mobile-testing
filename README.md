# Maestro Mobile Test Automation

**WebdriverIO Native Demo App v2.2.0**

Automated mobile testing using Maestro with **Page Object Model (POM)** architecture, **Clean Code** principles, and **Reusable Flows** patterns.

---

## Table of Contents

1. [Project Structure](#project-structure)
2. [Installation](#installation)
3. [Running Tests](#running-tests)
4. [POM Architecture](#pom-architecture)
5. [Variable Rules](#variable-rules)
6. [Test Coverage](#test-coverage)
7. [Tips & Troubleshooting](#tips--troubleshooting)

---

## Project Structure

```
maestro-wdio-demo/
├── config/
│   └── config.yml                      # Global constants reference
│
├── pages/                              # PAGE OBJECTS (one folder per screen)
│   ├── drag/
│   │   ├── drag_piece_to_target.yml    # Drag action with env: PIECE_ID, TARGET_ID
│   │   └── navigate_to_drag.yml        # Navigate to drag screen
│   │
│   ├── forms/
│   │   ├── fill_text_input.yml         # Input text with env: INPUT_TEXT
│   │   ├── navigate_to_forms.yml       # Navigate to forms screen
│   │   ├── select_dropdown.yml         # Select dropdown with env: DROPDOWN_VALUE
│   │   ├── tap_active_button.yml       # Tap active button
│   │   └── toggle_switch.yml           # Toggle switch element
│   │
│   ├── home/
│   │   └── launch_app.yml              # Launch application
│   │
│   ├── login/
│   │   ├── clear_login_form.yml        # Clear login form
│   │   ├── fill_login_form.yml         # Fill login with env: EMAIL, PASSWORD
│   │   ├── fill_signup_form.yml        # Fill signup with env: EMAIL, PASSWORD, CONFIRM_PASSWORD
│   │   ├── navigate_to_login.yml       # Navigate to login screen
│   │   ├── switch_to_login_tab.yml     # Switch to login tab
│   │   └── switch_to_signup_tab.yml    # Switch to signup tab
│   │
│   └── swipe/
│       ├── navigate_to_swipe.yml       # Navigate to swipe screen
│       ├── scroll_page_down.yml        # Scroll page down
│       ├── scroll_page_up.yml          # Scroll page up
│       ├── swipe_carousel_left.yml     # Swipe carousel left
│       └── swipe_carousel_right.yml    # Swipe carousel right
│
├── tests/                              # TEST CASES (one file per screen)
│   ├── TC001_home_test.yml             # Home screen test
│   ├── TC002_login_test.yml            # Login & signup test
│   ├── TC003_forms_test.yml            # Form elements test
│   ├── TC004_swipe_test.yml            # Swipe & scroll gestures test
│   └── TC005_drag_test.yml             # Drag & drop puzzle test
│
├── run_all_tests.yml                   # Master test runner
└── README.md                           # This documentation
```

---

## Installation

### Prerequisites

- **OS**: macOS, Linux, or Windows
- **Emulator/Device**: Android emulator or iOS simulator
- **Node.js**: Recommended (optional)

### Step 1: Install Maestro

**macOS / Linux:**

```bash
curl -Ls "https://get.maestro.mobile.dev" | bash
```

**Windows (PowerShell):**

```powershell
Invoke-WebRequest -Uri "https://get.maestro.mobile.dev" -OutFile maestro-install.ps1
.\maestro-install.ps1
```

Verify installation:

```bash
maestro --version
```

### Step 2: Prepare Emulator/Device

Ensure emulator is running or device is connected:

```bash
adb devices
```

Output should display connected device(s).

### Step 3: Install APK Application

Download APK from: [WebdriverIO Native Demo App](https://github.com/webdriverio/native-demo-app/releases)

Then install:

```bash
adb install wdio-native-app-v2.2.0.apk
```

---

## Running Tests

### Run All Tests

```bash
maestro test run_all_tests.yml
```

### Run Specific Test

```bash
# Home screen test
maestro test tests/TC001_home_test.yml

# Login & signup test
maestro test tests/TC002_login_test.yml

# Form elements test
maestro test tests/TC003_forms_test.yml

# Swipe & scroll test
maestro test tests/TC004_swipe_test.yml

# Drag & drop test
maestro test tests/TC005_drag_test.yml
```

### Run with Custom Environment Variables

```bash
maestro test --env EMAIL=custom@test.com tests/TC002_login_test.yml
maestro test --env PASSWORD=CustomPass123 tests/TC002_login_test.yml
```

### Debug Mode (Interactive Inspector)

```bash
maestro studio
```

---

## POM Architecture

Page Object Model structure follows the **Separation of Concerns** principle:

- **`pages/`**: Contains reusable action flows (no assertions)
- **`tests/`**: Contains test orchestration with assertions

### Example of CORRECT Structure

**File: `pages/login/fill_login_form.yml`** (Actions only, no assertions)

```yaml
appId: com.wdiodemoapp
---
- tapOn:
    id: 'input-email'
- clearText
- inputText: ${EMAIL}
- tapOn:
    id: 'input-password'
- inputText: ${PASSWORD}
- hideKeyboard
```

**File: `tests/TC002_login_test.yml`** (Orchestration + Assertions)

```yaml
appId: com.wdiodemoapp
env:
  VALID_EMAIL: 'test@webdriver.io'
  VALID_PASSWORD: 'Test1234!'
---
- runFlow: ../pages/home/launch_app.yml
- assertVisible: 'home'

- runFlow: ../pages/login/navigate_to_login.yml
- runFlow:
    file: ../pages/login/fill_login_form.yml
    env:
      EMAIL: '${VALID_EMAIL}'
      PASSWORD: '${VALID_PASSWORD}'
- assertVisible: 'Login Success'
```

### Benefits of This Structure

| Aspect              | Benefit                                              |
| ------------------- | ---------------------------------------------------- |
| **Reusability**     | Page flows can be called from multiple tests         |
| **Maintainability** | UI changes only need to be updated in one place      |
| **Readability**     | Test cases focus on scenarios, not technical details |
| **Scalability**     | Easy to add new tests                                |

---

## Variable Rules

### When Can You Use `${VARIABLE}`

| Context              | Allowed? | Example                        |
| -------------------- | -------- | ------------------------------ |
| **inputText value**  | Yes      | `inputText: ${EMAIL}`          |
| **Text label**       | Yes      | `text: "${EXPECTED_ERROR}"`    |
| **Partial ID**       | Yes      | `id: "tab-${TAB_NAME}"`        |
| **When expressions** | No       | `when: true: ${action == 'x'}` |
| **Comparisons**      | No       | `${x > 5}`, `${y == true}`     |

### CORRECT Usage

```yaml
# OK: Variable in value
- inputText: ${USERNAME}

# OK: Variable for simple conditions (ensure it's in env)
- runFlow:
    file: ../pages/login/fill_login_form.yml
    env:
      EMAIL: ${USER_EMAIL}

# WRONG: Expression evaluation
- runFlow:
    when:
      true: ${someVariable == 'value'} # ERROR!
```

---

## Test Coverage

| No  | Test File | Screen    | Description                     | Status |
| --- | --------- | --------- | ------------------------------- | ------ |
| 1   | TC001     | Home      | Smoke test - Launch application | Ready  |
| 2   | TC002     | Login     | Login & signup functionality    | Ready  |
| 3   | TC003     | Forms     | Fill form, dropdown, toggle     | Ready  |
| 4   | TC004     | Swipe     | Swipe carousel & scroll         | Ready  |
| 5   | TC005     | Drag      | Drag & drop puzzle pieces       | Ready  |
|     |           | **Total** | **5 Test Suites**               |        |

---

## Tips & Troubleshooting

### 1. Finding the Correct Test ID

Use Maestro Studio to inspect elements:

```bash
maestro studio
```

Steps:

1. Open Maestro Studio
2. Tap the element on screen that you want to inspect
3. Check the accessibility ID or text label
4. Update `id:` or `text:` in the relevant page object file

### 2. Selector Not Found

**Solutions:**

- Use `text:` if the label is unique and stable
- Use `id:` for elements with clear test ID
- Combine both if needed to disambiguate
- Verify element is visible on screen (use screenshot)

### 3. Test is Slow or Times Out

**Tips:**

- Reduce default timeout value if it's too long
- Ensure emulator has sufficient resources
- Check adb connection is not lost
- Restart emulator if needed

### 4. Variable Not Substituted

**Common causes:**

- Variable used in context that doesn't support expressions (`when:`, `if:`)
- Variable not defined in `env:` block
- Typo in variable name

**How to fix:**

```yaml
# Ensure variable is passed in env
- runFlow:
    file: ../pages/login/fill_login_form.yml
    env:
      EMAIL: 'test@example.com' # Must be here
```

### 5. Clean Application State

If tests repeatedly fail, reset application state:

```bash
adb uninstall com.wdiodemoapp
adb install wdio-native-app-v2.2.0.apk
```

---

## Best Practices

### Naming Convention

- Test file: `TC[XXX]_[screen]_test.yml`
- Page flow: `[action]_[object].yml`
- Variable: `UPPERCASE_WITH_UNDERSCORES`

### Code Organization

- **1 folder** = **1 screen** → easy to navigate
- **1 file** = **1 action** → easy to maintain
- **env in test** → data separated from logic
- **Screenshot per TC** → documentation of results

### Performance

- Minimize swipe/scroll commands
- Reuse flows as much as possible
- Batch related assertions
- Use explicit waits when needed

---

## Support & Resources

- **Maestro Documentation**: https://maestro.mobile.dev
- **WebdriverIO Demo App**: https://github.com/webdriverio/native-demo-app
- **Maestro Studio**: For interactive debugging
- **GitHub Issues**: Report bugs or request features

---

**Last Updated:** May 1, 2026  
**Maestro Version:** Latest  
**App Version:** v2.2.0
