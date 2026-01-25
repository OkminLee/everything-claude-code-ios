---
name: swift-tdd-workflow
description: Use this skill when writing new features, fixing bugs, or refactoring code. Enforces test-driven development with XCTest including unit, integration, and XCUITest E2E tests.
---

# Swift Test-Driven Development Workflow

This skill ensures all iOS development follows TDD principles with comprehensive test coverage.

## When to Activate

- Writing new features or functionality
- Fixing bugs or issues
- Refactoring existing code
- Adding API integrations
- Creating new SwiftUI views or UIKit components

## Core Principles

### 1. Tests BEFORE Code
ALWAYS write tests first, then implement code to make tests pass.

### 2. Coverage Requirements
- Minimum 80% coverage (unit + integration + UI)
- All edge cases covered
- Error scenarios tested
- Boundary conditions verified

### 3. Test Types

#### Unit Tests (XCTest)
- Individual functions and methods
- ViewModels and business logic
- Pure functions
- Helpers and utilities

#### Integration Tests
- API client operations
- Database operations
- Service interactions
- External API calls

#### UI Tests (XCUITest)
- Critical user flows
- Complete workflows
- Screen navigation
- UI interactions

## TDD Workflow Steps

### Step 1: Write User Stories
```
As a [role], I want to [action], so that [benefit]

Example:
As a user, I want to search for markets semantically,
so that I can find relevant markets even without exact keywords.
```

### Step 2: Generate Test Cases
For each user story, create comprehensive test cases:

```swift
import XCTest
@testable import MyApp

final class MarketSearchServiceTests: XCTestCase {

    var sut: MarketSearchService!
    var mockAPIClient: MockAPIClient!

    override func setUp() {
        super.setUp()
        mockAPIClient = MockAPIClient()
        sut = MarketSearchService(apiClient: mockAPIClient)
    }

    override func tearDown() {
        sut = nil
        mockAPIClient = nil
        super.tearDown()
    }

    func test_search_returnsRelevantMarkets() async throws {
        // Test implementation
    }

    func test_search_handlesEmptyQueryGracefully() async throws {
        // Test edge case
    }

    func test_search_fallsBackToLocalSearch_whenNetworkUnavailable() async throws {
        // Test fallback behavior
    }

    func test_search_sortsByRelevanceScore() async throws {
        // Test sorting logic
    }
}
```

### Step 3: Run Tests (They Should Fail)
```bash
xcodebuild test -scheme MyApp -destination 'platform=iOS Simulator,name=iPhone 15'
# Tests should fail - we haven't implemented yet
```

### Step 4: Implement Code
Write minimal code to make tests pass:

```swift
// Implementation guided by tests
final class MarketSearchService {
    private let apiClient: APIClientProtocol

    init(apiClient: APIClientProtocol) {
        self.apiClient = apiClient
    }

    func search(query: String) async throws -> [Market] {
        // Implementation here
    }
}
```

### Step 5: Run Tests Again
```bash
xcodebuild test -scheme MyApp -destination 'platform=iOS Simulator,name=iPhone 15'
# Tests should now pass
```

### Step 6: Refactor
Improve code quality while keeping tests green:
- Remove duplication
- Improve naming
- Optimize performance
- Enhance readability

### Step 7: Verify Coverage
```bash
xcodebuild test -scheme MyApp \
  -destination 'platform=iOS Simulator,name=iPhone 15' \
  -enableCodeCoverage YES
# Verify 80%+ coverage achieved
```

## Testing Patterns

### Unit Test Pattern (XCTest)

```swift
import XCTest
@testable import MyApp

final class MarketViewModelTests: XCTestCase {

    var sut: MarketViewModel!
    var mockService: MockMarketService!

    override func setUp() {
        super.setUp()
        mockService = MockMarketService()
        sut = MarketViewModel(service: mockService)
    }

    override func tearDown() {
        sut = nil
        mockService = nil
        super.tearDown()
    }

    // MARK: - Loading State

    func test_loadMarkets_setsLoadingTrue_initially() async {
        // Given
        mockService.delay = 0.1

        // When
        Task { await sut.loadMarkets() }

        // Then
        try? await Task.sleep(nanoseconds: 50_000_000) // 0.05s
        XCTAssertTrue(sut.isLoading)
    }

    func test_loadMarkets_setsLoadingFalse_afterCompletion() async {
        // Given
        mockService.marketsToReturn = [.preview]

        // When
        await sut.loadMarkets()

        // Then
        XCTAssertFalse(sut.isLoading)
    }

    // MARK: - Success Cases

    func test_loadMarkets_updatesMarkets_onSuccess() async {
        // Given
        let expectedMarkets = [Market.preview]
        mockService.marketsToReturn = expectedMarkets

        // When
        await sut.loadMarkets()

        // Then
        XCTAssertEqual(sut.markets, expectedMarkets)
    }

    // MARK: - Error Cases

    func test_loadMarkets_setsError_onFailure() async {
        // Given
        mockService.errorToThrow = NetworkError.noConnection

        // When
        await sut.loadMarkets()

        // Then
        XCTAssertNotNil(sut.error)
    }

    func test_loadMarkets_clearsMarkets_onFailure() async {
        // Given
        sut.markets = [.preview]
        mockService.errorToThrow = NetworkError.noConnection

        // When
        await sut.loadMarkets()

        // Then
        XCTAssertTrue(sut.markets.isEmpty)
    }
}
```

### Async Test Pattern

```swift
func test_asyncOperation_completesSuccessfully() async throws {
    // Given
    let expectation = XCTestExpectation(description: "Operation completes")
    var result: [Market]?

    // When
    Task {
        result = try await sut.fetchMarkets()
        expectation.fulfill()
    }

    // Then
    await fulfillment(of: [expectation], timeout: 5.0)
    XCTAssertNotNil(result)
    XCTAssertFalse(result!.isEmpty)
}

// Or using async/await directly
func test_asyncOperation_throws_onError() async {
    // Given
    mockService.errorToThrow = APIError.unauthorized

    // When/Then
    do {
        _ = try await sut.fetchMarkets()
        XCTFail("Expected error to be thrown")
    } catch {
        XCTAssertTrue(error is APIError)
    }
}
```

### UI Test Pattern (XCUITest)

```swift
import XCTest

final class MarketListUITests: XCTestCase {

    var app: XCUIApplication!

    override func setUp() {
        super.setUp()
        continueAfterFailure = false

        app = XCUIApplication()
        app.launchArguments = ["--uitesting"]
        app.launch()
    }

    override func tearDown() {
        app = nil
        super.tearDown()
    }

    func test_marketList_displaysMarkets() {
        // Given - App launched with market list visible

        // When - Navigate to markets tab
        app.tabBars.buttons["Markets"].tap()

        // Then - Market list is displayed
        let marketList = app.collectionViews["marketList"]
        XCTAssertTrue(marketList.waitForExistence(timeout: 5))
        XCTAssertGreaterThan(marketList.cells.count, 0)
    }

    func test_searchMarkets_filtersResults() {
        // Given
        app.tabBars.buttons["Markets"].tap()
        let searchField = app.searchFields["Search markets"]
        XCTAssertTrue(searchField.waitForExistence(timeout: 5))

        // When
        searchField.tap()
        searchField.typeText("election")

        // Then
        let marketList = app.collectionViews["marketList"]
        let timeout: TimeInterval = 5

        // Wait for filtered results
        let predicate = NSPredicate { _, _ in
            marketList.cells.count > 0 && marketList.cells.count < 10
        }
        let expectation = XCTNSPredicateExpectation(predicate: predicate, object: nil)
        wait(for: [expectation], timeout: timeout)
    }

    func test_tapMarket_navigatesToDetail() {
        // Given
        app.tabBars.buttons["Markets"].tap()
        let marketList = app.collectionViews["marketList"]
        XCTAssertTrue(marketList.waitForExistence(timeout: 5))

        // When
        let firstMarket = marketList.cells.firstMatch
        firstMarket.tap()

        // Then
        let detailView = app.navigationBars["Market Details"]
        XCTAssertTrue(detailView.waitForExistence(timeout: 5))
    }

    func test_createMarket_showsSuccessMessage() {
        // Given
        app.tabBars.buttons["Create"].tap()

        // When
        let nameField = app.textFields["marketName"]
        nameField.tap()
        nameField.typeText("Test Market")

        let descriptionField = app.textViews["marketDescription"]
        descriptionField.tap()
        descriptionField.typeText("Test description")

        app.buttons["Create Market"].tap()

        // Then
        let successMessage = app.staticTexts["Market created successfully"]
        XCTAssertTrue(successMessage.waitForExistence(timeout: 5))
    }
}
```

## Test File Organization

```
MyApp/
├── Sources/
│   ├── Features/
│   │   ├── Markets/
│   │   │   ├── Views/
│   │   │   ├── ViewModels/
│   │   │   └── Services/
│   │   └── Auth/
│   └── Core/
├── Tests/
│   ├── MyAppTests/                    # Unit tests
│   │   ├── Features/
│   │   │   ├── Markets/
│   │   │   │   ├── MarketViewModelTests.swift
│   │   │   │   └── MarketServiceTests.swift
│   │   │   └── Auth/
│   │   ├── Core/
│   │   └── Mocks/
│   │       ├── MockAPIClient.swift
│   │       └── MockMarketService.swift
│   └── MyAppUITests/                  # UI tests
│       ├── MarketListUITests.swift
│       ├── AuthUITests.swift
│       └── Helpers/
│           └── XCUIApplication+Extensions.swift
```

## Mocking Patterns

### Protocol-Based Mocking

```swift
// Protocol
protocol MarketServiceProtocol {
    func fetchMarkets() async throws -> [Market]
    func search(query: String) async throws -> [Market]
}

// Real implementation
final class MarketService: MarketServiceProtocol {
    func fetchMarkets() async throws -> [Market] {
        // Real API call
    }

    func search(query: String) async throws -> [Market] {
        // Real API call
    }
}

// Mock implementation
final class MockMarketService: MarketServiceProtocol {
    var marketsToReturn: [Market] = []
    var errorToThrow: Error?
    var delay: TimeInterval = 0

    func fetchMarkets() async throws -> [Market] {
        if delay > 0 {
            try await Task.sleep(nanoseconds: UInt64(delay * 1_000_000_000))
        }

        if let error = errorToThrow {
            throw error
        }

        return marketsToReturn
    }

    func search(query: String) async throws -> [Market] {
        if let error = errorToThrow {
            throw error
        }

        return marketsToReturn.filter { $0.name.contains(query) }
    }
}
```

### Network Mock

```swift
final class MockURLProtocol: URLProtocol {
    static var mockResponses: [URL: (Data, HTTPURLResponse)] = [:]

    override class func canInit(with request: URLRequest) -> Bool {
        true
    }

    override class func canonicalRequest(for request: URLRequest) -> URLRequest {
        request
    }

    override func startLoading() {
        guard let url = request.url,
              let (data, response) = MockURLProtocol.mockResponses[url] else {
            client?.urlProtocol(self, didFailWithError: URLError(.badURL))
            return
        }

        client?.urlProtocol(self, didReceive: response, cacheStoragePolicy: .notAllowed)
        client?.urlProtocol(self, didLoad: data)
        client?.urlProtocolDidFinishLoading(self)
    }

    override func stopLoading() {}
}

// Usage in tests
override func setUp() {
    super.setUp()

    let configuration = URLSessionConfiguration.ephemeral
    configuration.protocolClasses = [MockURLProtocol.self]
    let session = URLSession(configuration: configuration)

    sut = APIClient(session: session)
}
```

### SwiftUI View Testing (ViewInspector)

```swift
import XCTest
import ViewInspector
@testable import MyApp

extension MarketCard: Inspectable {}

final class MarketCardTests: XCTestCase {

    func test_marketCard_displaysName() throws {
        // Given
        let market = Market.preview

        // When
        let view = MarketCard(market: market)
        let name = try view.inspect().find(text: market.name)

        // Then
        XCTAssertNotNil(name)
    }

    func test_marketCard_displaysStatusBadge() throws {
        // Given
        let market = Market.preview

        // When
        let view = MarketCard(market: market)
        let badge = try view.inspect().find(StatusBadge.self)

        // Then
        XCTAssertNotNil(badge)
    }
}
```

## Test Coverage Verification

### Xcode Coverage Settings
1. Edit Scheme → Test → Options
2. Enable "Code Coverage"
3. Select targets to measure

### Minimum Coverage Thresholds

```swift
// Package.swift or xcconfig
// Set minimum coverage: 80%
```

### Coverage Report
```bash
# Generate coverage report
xcodebuild test \
  -scheme MyApp \
  -destination 'platform=iOS Simulator,name=iPhone 15' \
  -enableCodeCoverage YES \
  -resultBundlePath TestResults.xcresult

# View coverage
xcrun xccov view --report TestResults.xcresult
```

## Common Testing Mistakes to Avoid

### ❌ WRONG: Testing Implementation Details
```swift
// Don't test internal state
func test_viewModel_hasCorrectInternalState() {
    XCTAssertEqual(sut.internalState, .loading)
}
```

### ✅ CORRECT: Test Observable Behavior
```swift
// Test what users see
func test_viewModel_showsLoadingIndicator() {
    XCTAssertTrue(sut.isLoading)
}
```

### ❌ WRONG: Brittle Selectors
```swift
// Breaks easily
app.buttons.element(boundBy: 2).tap()
```

### ✅ CORRECT: Accessibility Identifiers
```swift
// Resilient to changes
app.buttons["submitButton"].tap()

// In SwiftUI view
Button("Submit") { }
    .accessibilityIdentifier("submitButton")
```

### ❌ WRONG: Shared Test State
```swift
// Tests depend on each other
static var createdMarket: Market?

func test_createMarket() {
    Self.createdMarket = createMarket()
}

func test_updateMarket() {
    updateMarket(Self.createdMarket!)  // Depends on previous test!
}
```

### ✅ CORRECT: Independent Tests
```swift
// Each test sets up its own data
func test_createMarket() {
    let market = createTestMarket()
    // Test logic
}

func test_updateMarket() {
    let market = createTestMarket()
    updateMarket(market)
    // Test logic
}
```

## Continuous Testing

### Watch Mode During Development
```bash
# Using xcodebuild with repeat
while true; do
    xcodebuild test -scheme MyApp -destination 'platform=iOS Simulator,name=iPhone 15'
    fswatch -1 Sources Tests
done
```

### Pre-Commit Hook
```bash
#!/bin/bash
# .git/hooks/pre-commit

echo "Running tests..."
xcodebuild test -scheme MyApp -destination 'platform=iOS Simulator,name=iPhone 15' -quiet

if [ $? -ne 0 ]; then
    echo "Tests failed. Commit aborted."
    exit 1
fi
```

### CI/CD Integration (GitHub Actions)
```yaml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: macos-14
    steps:
      - uses: actions/checkout@v4

      - name: Select Xcode
        run: sudo xcode-select -s /Applications/Xcode_15.2.app

      - name: Run Tests
        run: |
          xcodebuild test \
            -scheme MyApp \
            -destination 'platform=iOS Simulator,name=iPhone 15' \
            -enableCodeCoverage YES \
            -resultBundlePath TestResults.xcresult

      - name: Upload Coverage
        uses: codecov/codecov-action@v3
        with:
          xcode: true
          xcode_archive_path: TestResults.xcresult
```

## Best Practices

1. **Write Tests First** - Always TDD
2. **One Assert Per Test** - Focus on single behavior
3. **Descriptive Test Names** - `test_method_condition_expectation`
4. **Given-When-Then** - Clear test structure
5. **Mock External Dependencies** - Isolate unit tests
6. **Test Edge Cases** - nil, empty, large, boundary
7. **Test Error Paths** - Not just happy paths
8. **Keep Tests Fast** - Unit tests < 100ms each
9. **Clean Up After Tests** - tearDown properly
10. **Review Coverage Reports** - Identify gaps

## Success Metrics

- 80%+ code coverage achieved
- All tests passing (green)
- No skipped or disabled tests
- Fast test execution (< 60s for unit tests)
- UI tests cover critical user flows
- Tests catch bugs before production

---

**Remember**: Tests are not optional. They are the safety net that enables confident refactoring, rapid development, and production reliability.
