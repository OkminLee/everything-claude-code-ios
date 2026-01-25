---
description: Safely identify and remove dead code with test verification. Uses Periphery for Swift dead code analysis, categorizes findings by severity, and verifies with tests.
---

# Swift Refactor Clean

Safely identify and remove dead code with test verification for iOS projects.

This command invokes the **swift-refactor-cleaner** agent.

## What This Command Does

1. **Analyze Dead Code** - Run Periphery to find unused Swift code
2. **Generate Report** - Categorize findings by severity
3. **Propose Safe Deletions** - Start with low-risk items
4. **Verify with Tests** - Run tests before and after each change
5. **Rollback on Failure** - Revert if tests fail

## Process

1. Run dead code analysis tools:
   - **Periphery**: Find unused code, protocols, functions
   - **SPM**: `swift package diagnose-dependencies`
   - **Xcode**: Build with `-Wunused` warnings

2. Generate report in `.reports/dead-code-analysis.md`

3. Categorize findings by severity:
   - **SAFE**: Unused private methods, test utilities
   - **CAUTION**: Unused public APIs, ViewModels
   - **DANGER**: Protocol extensions, @objc code, App entry points

4. Propose safe deletions only

5. Before each deletion:
   - Run full test suite
   - Verify tests pass
   - Apply change
   - Re-run tests
   - Rollback if tests fail

6. Show summary of cleaned items

## Running Periphery

### Installation

```bash
# Homebrew
brew install peripheryapp/periphery/periphery

# Or build from source
git clone https://github.com/peripheryapp/periphery
cd periphery && make install
```

### Configuration

Create `.periphery.yml` in project root:

```yaml
# .periphery.yml
project: MyApp.xcodeproj
schemes:
  - MyApp
targets:
  - MyApp
  - MyAppTests

# Exclude patterns
exclude:
  - "*.generated.swift"
  - "**/Preview Content/**"
  - "**/*+Preview.swift"

# Retain patterns (false positives)
retain_public: false
retain_objc_accessible: true
retain_unused_protocol_func_params: true

# Output
format: json
quiet: false
```

### Running Analysis

```bash
# Full analysis
periphery scan --project MyApp.xcodeproj --schemes MyApp --targets MyApp

# With JSON output for parsing
periphery scan --format json > dead-code-report.json

# SPM project
periphery scan --setup

# Quick scan (faster, less accurate)
periphery scan --skip-build
```

## Report Format

```markdown
# Dead Code Analysis Report

Generated: 2024-01-23
Project: MyApp
Tool: Periphery 2.18.0

## Summary

| Category | Count |
|----------|-------|
| Unused Classes | 5 |
| Unused Functions | 12 |
| Unused Protocols | 3 |
| Unused Properties | 8 |
| **Total** | **28** |

---

## 🟢 SAFE to Remove (15)

### Unused Private Methods

1. `HomeViewModel.swift:45` - `private func legacyFetch()`
   - Not called anywhere
   - Safe to remove

2. `UserService.swift:89` - `private var tempCache: [String: Any]`
   - Declared but never used
   - Safe to remove

### Unused Test Utilities

3. `TestHelpers.swift:12` - `func createMockUser()`
   - No tests using this helper
   - Verify no test needs it, then remove

---

## 🟡 CAUTION (10)

### Unused Public APIs

1. `NetworkClient.swift:23` - `public func fetchLegacy()`
   - May be used by external modules
   - Check all targets before removing

2. `Analytics.swift:56` - `AnalyticsEventType.pageView`
   - Enum case appears unused
   - May be used via string interpolation

### Unused ViewModels

3. `SettingsViewModel.swift` - Entire file
   - SettingsView may instantiate via @StateObject
   - Check SwiftUI view bindings

---

## 🔴 DANGER - Do Not Auto-Remove (3)

### @objc Accessible

1. `AppDelegate.swift:34` - `@objc func handleDeepLink()`
   - Called by Objective-C runtime
   - May be triggered externally

### Protocol Extensions

2. `Codable+Extensions.swift:12` - `extension Encodable`
   - Used via protocol conformance
   - Periphery may not detect usage

### Entry Points

3. `main.swift` - `@main`
   - App entry point
   - Never remove
```

## Severity Guidelines

### 🟢 SAFE to Remove

- `private` methods not called within file
- `fileprivate` members not used in file
- Commented-out code blocks
- Unused imports
- Dead test utilities
- Unused stub/mock classes in test target

### 🟡 CAUTION

- `internal` APIs (may be used in other files)
- `public` APIs (may be used by other targets/modules)
- Protocol conformance methods
- ViewModels (SwiftUI instantiation may not be detected)
- Extension methods
- Computed properties

### 🔴 DANGER - Never Auto-Remove

- `@objc` accessible code
- `@IBAction` / `@IBOutlet`
- Protocol requirements
- `@main` / entry points
- NotificationCenter selectors
- Codable implementations
- Equatable/Hashable implementations

## Verification Process

```bash
# Step 1: Run tests before change
xcodebuild test -scheme MyApp -destination 'platform=iOS Simulator,name=iPhone 16'

# Step 2: Remove dead code

# Step 3: Build to check compilation
xcodebuild build -scheme MyApp

# Step 4: Run tests after change
xcodebuild test -scheme MyApp -destination 'platform=iOS Simulator,name=iPhone 16'

# Step 5: If tests fail, revert
git checkout -- .
```

## SPM Dependency Analysis

```bash
# Check for unused dependencies
swift package diagnose-dependencies

# Update to latest compatible versions
swift package update

# Show dependency graph
swift package show-dependencies --format json
```

## Asset Catalog Cleanup

Find unused images and colors:

```bash
# Find all asset references in code
grep -rh "Image(\|UIImage(named:\|Color(\"\|UIColor(named:" --include="*.swift" Sources/

# Compare with Assets.xcassets contents
find . -path "*/Assets.xcassets/*" -name "*.imageset" -o -name "*.colorset"

# Use FengNiao tool for automated detection
fengniao --project . --exclude Pods
```

## Integration with CI

```yaml
# GitHub Actions
- name: Dead Code Analysis
  run: |
    periphery scan \
      --project MyApp.xcodeproj \
      --schemes MyApp \
      --format json > dead-code.json

    # Fail if new dead code introduced
    COUNT=$(jq length dead-code.json)
    if [ "$COUNT" -gt "$ALLOWED_DEAD_CODE" ]; then
      echo "❌ Dead code count ($COUNT) exceeds limit ($ALLOWED_DEAD_CODE)"
      exit 1
    fi
```

## Best Practices

**DO:**
- ✅ Run full test suite before and after each removal
- ✅ Remove one category at a time (all unused privates first)
- ✅ Commit after each successful removal batch
- ✅ Review CAUTION items manually
- ✅ Keep `.periphery.yml` in version control

**DON'T:**
- ❌ Remove code without test verification
- ❌ Auto-remove DANGER category items
- ❌ Remove multiple unrelated items in one commit
- ❌ Ignore Periphery false positives (add to retain list)
- ❌ Remove @objc or protocol code without manual review

## Integration with Other Commands

- Use `/swift-review` to verify cleaned code quality
- Use `/swift-tdd` to add tests before removing code
- Use `/xcode-build-fix` if removal causes build errors

## Related Agents

This command invokes the `swift-refactor-cleaner` agent located at:
`~/.claude/agents/swift-refactor-cleaner.md`

**Never delete code without running tests first!**
