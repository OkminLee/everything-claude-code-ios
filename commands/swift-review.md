---
description: Comprehensive security and quality review for Swift/iOS code changes. Checks for memory leaks, concurrency issues, and iOS security vulnerabilities.
---

# Swift Code Review

Comprehensive security and quality review of uncommitted changes for iOS projects.

This command invokes the **swift-code-reviewer** agent.

## Process

1. Get changed files: `git diff --name-only HEAD`

2. For each changed Swift file, check for:

**Security Issues (CRITICAL):**
- Sensitive data in UserDefaults (use Keychain)
- Hardcoded credentials, API keys, tokens
- App Transport Security (ATS) exceptions without justification
- Missing certificate pinning for financial/health apps
- Biometric auth bypass vulnerabilities
- Clipboard exposure of sensitive data
- Debug code in production (#if DEBUG missing)
- WebView JavaScript injection risks

**Memory Management (CRITICAL):**
- Retain cycles in closures (missing `[weak self]`)
- Delegates not declared as `weak`
- NotificationCenter observers not removed
- Timer not invalidated on deinit
- URLSession tasks not cancelled
- Combine: `sink` without storing cancellable
- Combine: `assign(to:on:)` creating retain cycle

**Code Quality (HIGH):**
- Force unwrapping (`!`) without guard/if-let
- Functions > 40 lines
- Files > 500 lines
- Nesting depth > 4 levels
- Missing access control (private by default)
- `print()` statements in production code
- TODO/FIXME comments
- Missing documentation for public APIs
- Empty catch blocks
- `try!` or `try?` without justification

**Concurrency (HIGH):**
- UI updates not on main thread
- Missing `@MainActor` for ViewModel
- Data races in shared mutable state
- Actor isolation violations
- Sendable compliance issues
- Task cancellation not handled

**Best Practices (MEDIUM):**
- Not using struct for data models (prefer value types)
- Missing SwiftUI previews for views
- Accessibility identifiers missing
- Missing tests for new code
- Hardcoded strings (should be localized)
- Magic numbers without constants

3. Generate report with:
   - Severity: 🔴 CRITICAL, 🟡 HIGH, 🟢 MEDIUM
   - File location: `path/to/file.swift:lineNumber`
   - Issue description
   - Code example of the problem
   - Suggested fix with code

4. Block commit if CRITICAL issues found

## Report Format

```
## Code Review Report

### 🔴 CRITICAL (Must Fix)

#### 1. Retain Cycle in Closure
**File**: `Features/Home/HomeViewModel.swift:45`

**Issue**: Closure captures `self` strongly, causing memory leak.

**Current**:
```swift
networkService.fetch { result in
    self.items = result  // Strong capture
}
```

**Fix**:
```swift
networkService.fetch { [weak self] result in
    self?.items = result
}
```

---

### 🟡 HIGH (Should Fix)

#### 1. Force Unwrap
**File**: `Models/User.swift:23`

**Issue**: Force unwrapping can crash if value is nil.

**Current**:
```swift
let name = user.name!
```

**Fix**:
```swift
guard let name = user.name else { return }
```

---

### 🟢 MEDIUM (Consider)

#### 1. Missing Accessibility
**File**: `Views/LoginView.swift:67`

**Issue**: Button missing accessibility identifier for UI testing.

**Fix**:
```swift
Button("Login") { ... }
    .accessibilityIdentifier("login-button")
```
```

## Quick Checks

Run these before committing:

```bash
# Check for print statements
grep -rn "print(" --include="*.swift" Sources/

# Check for force unwraps
grep -rn "!" --include="*.swift" Sources/ | grep -v "//"

# Check for TODO/FIXME
grep -rn "TODO\|FIXME" --include="*.swift" Sources/

# Check for hardcoded secrets (basic)
grep -rn "api_key\|apiKey\|secret\|password" --include="*.swift" Sources/
```

## Integration with Other Commands

- Use `/swift-tdd` to add tests for flagged code
- Use `/swift-refactor` to address code quality issues
- Use `/xcode-build-fix` if refactoring causes build errors

## Related Agents

This command invokes the `swift-code-reviewer` agent located at:
`~/.claude/agents/swift-code-reviewer.md`

**Never approve code with security vulnerabilities or memory leaks!**
