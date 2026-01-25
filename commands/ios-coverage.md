---
description: Analyze test coverage and generate missing tests for iOS projects. Uses xcodebuild coverage and xccov to identify under-tested code and generate tests.
---

# iOS Test Coverage

Analyze test coverage and generate missing tests for iOS projects.

## What This Command Does

1. **Run Tests with Coverage** - Execute tests with code coverage enabled
2. **Analyze Coverage Report** - Parse xcresult bundle with xccov
3. **Identify Gaps** - Find files below 80% threshold
4. **Generate Missing Tests** - Create unit/integration tests
5. **Verify and Report** - Show before/after metrics

## Process

1. Run tests with coverage:
   ```bash
   xcodebuild test \
     -scheme MyApp \
     -destination 'platform=iOS Simulator,name=iPhone 16' \
     -enableCodeCoverage YES \
     -resultBundlePath TestResults.xcresult
   ```

2. Analyze coverage report:
   ```bash
   xcrun xccov view --report TestResults.xcresult
   ```

3. Identify files below 80% coverage threshold

4. For each under-covered file:
   - Analyze untested code paths
   - Generate unit tests for functions
   - Generate integration tests for repositories
   - Identify edge cases to cover

5. Verify new tests pass

6. Show before/after coverage metrics

7. Ensure project reaches 80%+ overall coverage

## Running Coverage Analysis

### Xcode Project

```bash
# Run tests with coverage
xcodebuild test \
  -scheme MyApp \
  -destination 'platform=iOS Simulator,name=iPhone 16' \
  -enableCodeCoverage YES \
  -resultBundlePath TestResults.xcresult

# View summary report
xcrun xccov view --report TestResults.xcresult

# View JSON report (for parsing)
xcrun xccov view --report --json TestResults.xcresult > coverage.json

# View specific file coverage
xcrun xccov view --archive --file Sources/HomeViewModel.swift TestResults.xcresult
```

### Swift Package

```bash
# Run with coverage
swift test --enable-code-coverage

# Find coverage data
COVERAGE_PATH=$(swift test --show-codecov-path)

# Convert to report
xcrun llvm-cov report \
  .build/debug/MyPackagePackageTests.xctest/Contents/MacOS/MyPackagePackageTests \
  -instr-profile=$COVERAGE_PATH
```

## Coverage Report Format

```
╔══════════════════════════════════════════════════════════════╗
║                    Coverage Report                           ║
╠══════════════════════════════════════════════════════════════╣
║ Overall Coverage: 72.4% (Target: 80%)                        ║
║ Status: ❌ BELOW THRESHOLD                                   ║
╚══════════════════════════════════════════════════════════════╝

## Coverage by Target

| Target | Coverage | Status |
|--------|----------|--------|
| MyApp | 68.2% | ❌ |
| MyAppTests | 100% | ✅ |

## Files Below Threshold (<80%)

| File | Coverage | Lines Missed | Priority |
|------|----------|--------------|----------|
| HomeViewModel.swift | 45.2% | 28/51 | 🔴 HIGH |
| NetworkClient.swift | 62.1% | 19/50 | 🟡 MEDIUM |
| UserService.swift | 78.3% | 5/23 | 🟢 LOW |

## Untested Code Paths

### HomeViewModel.swift (45.2%)

**Untested Functions:**
1. `fetchItems()` - Line 23-45
   - Error handling path not tested
   - Empty result case not tested

2. `deleteItem(id:)` - Line 67-89
   - Entire function untested

3. `refreshData()` - Line 102-115
   - Cancellation path not tested

**Recommended Tests:**
```swift
@Test func fetchItems_onError_showsAlert() async {
    // Test error handling path
}

@Test func fetchItems_withEmptyResult_showsEmptyState() async {
    // Test empty result case
}

@Test func deleteItem_removesFromList() async {
    // Test delete functionality
}
```
```

## Coverage Thresholds

| Category | Target | Critical |
|----------|--------|----------|
| Overall Project | 80% | - |
| ViewModels | 85% | - |
| UseCases | 90% | ✅ |
| Business Logic | 95% | ✅ |
| UI Code (Views) | 50% | - |
| Generated Code | Skip | - |

**Critical Code** (100% required):
- Financial calculations
- Authentication logic
- Security-related code
- Core business rules

## Focus Areas

When generating tests, prioritize:

### Happy Path Scenarios
```swift
@Test func fetchItems_returnsItems() async throws {
    // Given
    mockRepo.items = [Item.stub()]

    // When
    await sut.fetchItems()

    // Then
    #expect(sut.items.count == 1)
}
```

### Error Handling
```swift
@Test func fetchItems_onNetworkError_showsError() async {
    // Given
    mockRepo.error = NetworkError.noConnection

    // When
    await sut.fetchItems()

    // Then
    #expect(sut.showingError)
    #expect(sut.errorMessage == "No connection")
}
```

### Edge Cases
```swift
// nil handling
@Test func user_withNilName_usesDefault() {
    let user = User(name: nil)
    #expect(user.displayName == "Unknown")
}

// Empty collections
@Test func fetchItems_withEmptyResult_showsEmptyState() async {
    mockRepo.items = []
    await sut.fetchItems()
    #expect(sut.showingEmptyState)
}

// Optional unwrapping
@Test func profile_withMissingAvatar_usesPlaceholder() {
    let profile = Profile(avatarURL: nil)
    #expect(profile.avatarImage == Image.placeholder)
}
```

### Boundary Conditions
```swift
@Test(arguments: [
    (input: 0, expected: "Zero"),
    (input: 1, expected: "One"),
    (input: Int.max, expected: "Max"),
    (input: -1, expected: "Negative")
])
func format_handlesEdgeCases(input: Int, expected: String) {
    #expect(sut.format(input) == expected)
}
```

## Excluding Files from Coverage

In Xcode scheme settings or xccov:

```bash
# Exclude patterns when viewing
xcrun xccov view --report TestResults.xcresult \
  --files-for-target MyApp \
  | grep -v "Generated\|Preview\|Mock"
```

Or configure in scheme:
1. Edit Scheme → Test → Options
2. Code Coverage → Gather coverage for: selected targets
3. Exclude: Generated files, Previews, Mocks

## CI/CD Integration

### GitHub Actions

```yaml
- name: Run Tests with Coverage
  run: |
    xcodebuild test \
      -scheme MyApp \
      -destination 'platform=iOS Simulator,name=iPhone 16' \
      -enableCodeCoverage YES \
      -resultBundlePath TestResults.xcresult

- name: Check Coverage Threshold
  run: |
    COVERAGE=$(xcrun xccov view --report --json TestResults.xcresult \
      | jq '.targets[] | select(.name == "MyApp.app") | .lineCoverage')

    if (( $(echo "$COVERAGE < 0.80" | bc -l) )); then
      echo "❌ Coverage ($COVERAGE) is below 80%"
      exit 1
    fi
    echo "✅ Coverage: $COVERAGE"

- name: Upload Coverage Report
  uses: actions/upload-artifact@v4
  with:
    name: coverage-report
    path: TestResults.xcresult
```

### Codecov Integration

```yaml
- name: Upload to Codecov
  run: |
    # Convert xcresult to lcov format
    xcrun llvm-cov export \
      -format=lcov \
      -instr-profile=$(find . -name "*.profdata") \
      MyApp > coverage.lcov

    bash <(curl -s https://codecov.io/bash) -f coverage.lcov
```

## Generating Missing Tests

When coverage is below threshold:

1. **Identify untested paths** using xccov line-by-line report
2. **Prioritize by risk**: Business logic > UI > Utilities
3. **Write focused tests**: One test per scenario
4. **Use Swift Testing** for new tests
5. **Run and verify** coverage improvement

## Best Practices

**DO:**
- ✅ Set coverage thresholds per module
- ✅ Focus on business logic coverage (90%+)
- ✅ Test error handling paths
- ✅ Test edge cases (nil, empty, bounds)
- ✅ Exclude generated/preview code

**DON'T:**
- ❌ Chase 100% coverage for UI code
- ❌ Write tests just to increase numbers
- ❌ Test trivial getters/setters
- ❌ Include test code in coverage

## Integration with Other Commands

- Use `/swift-tdd` to write tests for new code
- Use `/swift-review` to verify test quality
- Use `/xcuitest` for UI flow coverage

## Quick Commands

```bash
# Quick coverage check
xcodebuild test -scheme MyApp -enableCodeCoverage YES \
  -destination 'platform=iOS Simulator,name=iPhone 16' \
  -resultBundlePath Results.xcresult && \
  xcrun xccov view --report Results.xcresult

# Coverage for specific file
xcrun xccov view --archive \
  --file Sources/HomeViewModel.swift \
  Results.xcresult

# Export to JSON for analysis
xcrun xccov view --report --json Results.xcresult > coverage.json
```
