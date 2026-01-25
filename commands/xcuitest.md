---
description: Generate and run end-to-end tests with XCUITest. Creates test journeys using Page Object Model, runs tests, captures screenshots/videos, and manages flaky tests.
---

# XCUITest Command

This command invokes the **xcuitest-runner** agent to generate, maintain, and execute end-to-end UI tests for iOS apps.

## What This Command Does

1. **Generate Test Journeys** - Create XCUITest tests for user flows
2. **Run UI Tests** - Execute tests on simulators/devices
3. **Capture Artifacts** - Screenshots, videos on failures
4. **Manage Flaky Tests** - Quarantine unstable tests
5. **Generate Reports** - xcresult bundles for analysis

## When to Use

Use `/xcuitest` when:
- Testing critical user journeys (login, onboarding, payments)
- Verifying multi-step flows work end-to-end
- Testing UI interactions and navigation
- Validating accessibility
- Preparing for App Store submission
- Testing push notification handling

## How It Works

The xcuitest-runner agent will:

1. **Analyze user flow** and identify test scenarios
2. **Generate XCUITest** using Page Object Model (Screen Objects)
3. **Set up accessibility identifiers** if missing
4. **Run tests** on specified simulators
5. **Capture failures** with screenshots and videos
6. **Generate report** with results and artifacts

## Example Usage

```
User: /xcuitest Test the login and home flow

Agent (xcuitest-runner):
# E2E Test Generation: Login to Home Flow

## Test Scenario Identified

**User Journey:** Launch → Login → Home Screen

**Test Coverage:**
1. App launches successfully
2. Login screen appears
3. Enter credentials
4. Tap login button
5. Home screen loads
6. User name displayed

## Generated Screen Objects

```swift
// UITests/Screens/LoginScreen.swift
import XCTest

final class LoginScreen {
    private let app: XCUIApplication

    init(app: XCUIApplication) {
        self.app = app
    }

    // MARK: - Elements
    var emailField: XCUIElement { app.textFields["email-field"] }
    var passwordField: XCUIElement { app.secureTextFields["password-field"] }
    var loginButton: XCUIElement { app.buttons["login-button"] }
    var errorLabel: XCUIElement { app.staticTexts["error-label"] }

    // MARK: - Actions
    @discardableResult
    func typeEmail(_ email: String) -> Self {
        emailField.tap()
        emailField.typeText(email)
        return self
    }

    @discardableResult
    func typePassword(_ password: String) -> Self {
        passwordField.tap()
        passwordField.typeText(password)
        return self
    }

    @discardableResult
    func tapLogin() -> HomeScreen {
        loginButton.tap()
        return HomeScreen(app: app)
    }

    // MARK: - Assertions
    @discardableResult
    func assertErrorShown(_ message: String) -> Self {
        XCTAssertTrue(errorLabel.waitForExistence(timeout: 3))
        XCTAssertTrue(errorLabel.label.contains(message))
        return self
    }
}
```

```swift
// UITests/Screens/HomeScreen.swift
import XCTest

final class HomeScreen {
    private let app: XCUIApplication

    init(app: XCUIApplication) {
        self.app = app
    }

    // MARK: - Elements
    var welcomeLabel: XCUIElement { app.staticTexts["welcome-label"] }
    var profileButton: XCUIElement { app.buttons["profile-button"] }
    var settingsButton: XCUIElement { app.buttons["settings-button"] }

    // MARK: - Assertions
    @discardableResult
    func assertWelcomeShown(for userName: String) -> Self {
        XCTAssertTrue(welcomeLabel.waitForExistence(timeout: 5))
        XCTAssertTrue(welcomeLabel.label.contains(userName))
        return self
    }

    @discardableResult
    func assertLoaded() -> Self {
        XCTAssertTrue(welcomeLabel.waitForExistence(timeout: 5))
        return self
    }
}
```

## Generated Test Code

```swift
// UITests/Flows/LoginFlowTests.swift
import XCTest

final class LoginFlowTests: XCTestCase {
    var app: XCUIApplication!

    override func setUpWithError() throws {
        continueAfterFailure = false
        app = XCUIApplication()
        app.launchArguments = ["--uitesting", "--reset-state"]
        app.launch()
    }

    override func tearDownWithError() throws {
        // Capture screenshot on failure
        if let failureCount = testRun?.failureCount, failureCount > 0 {
            let screenshot = app.screenshot()
            let attachment = XCTAttachment(screenshot: screenshot)
            attachment.lifetime = .keepAlways
            add(attachment)
        }
        app.terminate()
    }

    func test_loginWithValidCredentials_showsHomeScreen() throws {
        // Given
        let loginScreen = LoginScreen(app: app)

        // When
        let homeScreen = loginScreen
            .typeEmail("test@example.com")
            .typePassword("password123")
            .tapLogin()

        // Then
        homeScreen.assertLoaded()
        homeScreen.assertWelcomeShown(for: "Test User")
    }

    func test_loginWithInvalidCredentials_showsError() throws {
        // Given
        let loginScreen = LoginScreen(app: app)

        // When
        loginScreen
            .typeEmail("invalid@example.com")
            .typePassword("wrongpassword")
            .tapLogin()

        // Then
        loginScreen.assertErrorShown("Invalid credentials")
    }

    func test_loginWithEmptyFields_showsValidationError() throws {
        // Given
        let loginScreen = LoginScreen(app: app)

        // When
        loginScreen.tapLogin()

        // Then
        loginScreen.assertErrorShown("Please enter email")
    }
}
```

## Running Tests

```bash
# Run UI tests
xcodebuild test \
  -scheme MyApp \
  -destination 'platform=iOS Simulator,name=iPhone 16' \
  -only-testing:MyAppUITests

Running 3 tests using 1 workers

  ✓ test_loginWithValidCredentials_showsHomeScreen (2.4s)
  ✓ test_loginWithInvalidCredentials_showsError (1.8s)
  ✓ test_loginWithEmptyFields_showsValidationError (1.2s)

3 passed (5.4s)

Artifacts generated:
- TestResults.xcresult
```

## Test Report

```
╔══════════════════════════════════════════════════════════════╗
║                    XCUITest Results                          ║
╠══════════════════════════════════════════════════════════════╣
║ Status:     ✅ ALL TESTS PASSED                              ║
║ Total:      3 tests                                          ║
║ Passed:     3 (100%)                                         ║
║ Failed:     0                                                ║
║ Flaky:      0                                                ║
║ Duration:   5.4s                                             ║
╚══════════════════════════════════════════════════════════════╝

Artifacts:
📸 Screenshots: Captured on failure
📹 Videos: Captured on failure
📊 Result Bundle: TestResults.xcresult

View report: open TestResults.xcresult
```

✅ UI test suite ready for CI/CD integration!
```

## Accessibility Identifiers

Ensure views have accessibility identifiers for reliable testing:

```swift
// In SwiftUI
TextField("Email", text: $email)
    .accessibilityIdentifier("email-field")

Button("Login") { /* ... */ }
    .accessibilityIdentifier("login-button")

// In UIKit
emailTextField.accessibilityIdentifier = "email-field"
loginButton.accessibilityIdentifier = "login-button"
```

## Handling UI Interruptions

Handle system alerts (permissions, notifications):

```swift
override func setUpWithError() throws {
    // ...
    addUIInterruptionMonitor(withDescription: "System Alert") { alert in
        if alert.buttons["Allow"].exists {
            alert.buttons["Allow"].tap()
            return true
        }
        if alert.buttons["OK"].exists {
            alert.buttons["OK"].tap()
            return true
        }
        return false
    }
}
```

## Flaky Test Management

```
⚠️  FLAKY TEST DETECTED: LoginFlowTests/test_loginWithValidCredentials

Test passed 7/10 runs (70% pass rate)

Common failure:
"Failed to find element: welcome-label (timeout: 5s)"

Recommended fixes:
1. Increase timeout: waitForExistence(timeout: 10)
2. Add explicit wait for network response
3. Check for animation completion
4. Verify launch arguments reset state properly

Quarantine recommendation:
```swift
func test_loginWithValidCredentials_showsHomeScreen() throws {
    try XCTSkipIf(ProcessInfo.processInfo.environment["SKIP_FLAKY"] == "true",
                  "Skipping flaky test - tracking in #ISSUE-123")
    // ...
}
```
```

## Running on Multiple Devices

```bash
# iPhone SE (small screen)
xcodebuild test \
  -scheme MyApp \
  -destination 'platform=iOS Simulator,name=iPhone SE (3rd generation)'

# iPhone 16 Pro Max (large screen)
xcodebuild test \
  -scheme MyApp \
  -destination 'platform=iOS Simulator,name=iPhone 16 Pro Max'

# iPad
xcodebuild test \
  -scheme MyApp \
  -destination 'platform=iOS Simulator,name=iPad Pro (12.9-inch)'
```

## CI/CD Integration

### GitHub Actions

```yaml
- name: Run UI Tests
  run: |
    xcodebuild test \
      -scheme MyApp \
      -destination 'platform=iOS Simulator,name=iPhone 16' \
      -only-testing:MyAppUITests \
      -resultBundlePath TestResults.xcresult

- name: Upload Test Results
  if: always()
  uses: actions/upload-artifact@v4
  with:
    name: uitest-results
    path: TestResults.xcresult
```

### Xcode Cloud

```yaml
# ci_post_xcodebuild.sh
if [ "$CI_XCODEBUILD_ACTION" == "test" ]; then
    xcrun xcresulttool export --path "$CI_RESULT_BUNDLE_PATH" \
        --type screenshots \
        --output-path "$CI_RESULT_BUNDLE_PATH/screenshots"
fi
```

## Critical Flows to Test

**🔴 CRITICAL (Must Always Pass):**
1. App launch and initial screen
2. User login/logout
3. Onboarding flow
4. Core feature navigation
5. Purchase/payment flows
6. Error state displays

**🟡 IMPORTANT:**
1. Deep link handling
2. Push notification taps
3. Background/foreground transitions
4. Accessibility compliance
5. Dark mode appearance
6. Dynamic Type support

## Best Practices

**DO:**
- ✅ Use Page Object Model for maintainability
- ✅ Use accessibility identifiers for selectors
- ✅ Wait for elements, not arbitrary delays
- ✅ Test critical user journeys end-to-end
- ✅ Handle UI interruptions (alerts, permissions)
- ✅ Reset app state in setUp

**DON'T:**
- ❌ Use sleep() for waits (use waitForExistence)
- ❌ Test implementation details
- ❌ Run tests against production servers
- ❌ Ignore flaky tests
- ❌ Over-test with UI tests (prefer unit tests)
- ❌ Hardcode text (use accessibility identifiers)

## Integration with Other Commands

- Use `/plan` to identify critical journeys to test
- Use `/swift-tdd` for unit tests (faster, more granular)
- Use `/xcuitest` for UI and user journey tests
- Use `/swift-review` to verify test quality

## Related Agents

This command invokes the `xcuitest-runner` agent located at:
`~/.claude/agents/xcuitest-runner.md`

## Quick Commands

```bash
# Run all UI tests
xcodebuild test -scheme MyApp -only-testing:MyAppUITests \
  -destination 'platform=iOS Simulator,name=iPhone 16'

# Run specific test class
xcodebuild test -scheme MyApp \
  -only-testing:MyAppUITests/LoginFlowTests \
  -destination 'platform=iOS Simulator,name=iPhone 16'

# Run with video recording
xcodebuild test -scheme MyApp \
  -destination 'platform=iOS Simulator,name=iPhone 16' \
  -enableCodeCoverage YES \
  -resultBundlePath TestResults.xcresult

# View result bundle
open TestResults.xcresult
```
