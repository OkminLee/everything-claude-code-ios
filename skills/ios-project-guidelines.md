---
name: ios-project-guidelines
description: iOS project-specific guidelines template for architecture, file structure, code patterns, testing, and deployment workflow.
---

# iOS Project Guidelines (Template)

This is a template for project-specific skills. Customize this for your iOS projects.

---

## When to Use

Reference this skill when working on the specific iOS project it's designed for. Project skills contain:
- Architecture overview
- File structure
- Code patterns
- Testing requirements
- Deployment workflow

---

## Architecture Overview

**Tech Stack:**
- **UI Framework**: SwiftUI (iOS 17+) / UIKit compatibility
- **Architecture**: MVVM + Clean Architecture
- **Networking**: URLSession + async/await
- **Persistence**: SwiftData / Core Data / UserDefaults
- **DI**: Factory pattern / Environment injection
- **Testing**: XCTest (Unit) + XCUITest (E2E)

**Layer Diagram:**
```
┌─────────────────────────────────────────────────────────────┐
│                      Presentation Layer                      │
│  SwiftUI Views + ViewModels                                  │
│  Handles UI and user interaction                             │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                       Domain Layer                           │
│  Use Cases + Entities + Repository Protocols                 │
│  Business logic, independent of frameworks                   │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                        Data Layer                            │
│  Repository Implementations + Data Sources                   │
│  API clients, local storage, external services               │
└─────────────────────────────────────────────────────────────┘
```

---

## File Structure

```
ProjectName/
├── App/
│   ├── ProjectNameApp.swift         # App entry point
│   ├── AppDelegate.swift            # UIKit lifecycle (if needed)
│   └── DependencyContainer.swift    # DI container
│
├── Features/                        # Feature modules
│   ├── Markets/
│   │   ├── Views/
│   │   │   ├── MarketListView.swift
│   │   │   └── MarketDetailView.swift
│   │   ├── ViewModels/
│   │   │   ├── MarketListViewModel.swift
│   │   │   └── MarketDetailViewModel.swift
│   │   ├── Models/
│   │   │   └── Market.swift
│   │   └── Services/
│   │       └── MarketService.swift
│   │
│   ├── Auth/
│   │   ├── Views/
│   │   ├── ViewModels/
│   │   └── Services/
│   │
│   └── Settings/
│       ├── Views/
│       └── ViewModels/
│
├── Core/                            # Shared infrastructure
│   ├── Network/
│   │   ├── APIClient.swift
│   │   ├── APIError.swift
│   │   └── Endpoints.swift
│   ├── Storage/
│   │   ├── KeychainManager.swift
│   │   └── UserDefaultsManager.swift
│   └── Extensions/
│       ├── Date+Formatting.swift
│       └── View+Extensions.swift
│
├── UI/                              # Reusable UI components
│   ├── Components/
│   │   ├── LoadingView.swift
│   │   ├── ErrorView.swift
│   │   └── EmptyStateView.swift
│   └── Styles/
│       ├── ButtonStyles.swift
│       └── Colors.swift
│
├── Resources/
│   ├── Assets.xcassets
│   ├── Localizable.strings
│   └── Info.plist
│
└── Tests/
    ├── UnitTests/
    │   ├── Features/
    │   └── Core/
    └── UITests/
        └── Features/
```

---

## Code Patterns

### ViewModel Pattern

```swift
import SwiftUI

@Observable
final class MarketListViewModel {
    // MARK: - State
    private(set) var markets: [Market] = []
    private(set) var isLoading = false
    private(set) var error: Error?

    // MARK: - Dependencies
    private let marketService: MarketServiceProtocol

    // MARK: - Init
    init(marketService: MarketServiceProtocol = MarketService()) {
        self.marketService = marketService
    }

    // MARK: - Actions
    @MainActor
    func loadMarkets() async {
        isLoading = true
        error = nil

        do {
            markets = try await marketService.fetchMarkets()
        } catch {
            self.error = error
        }

        isLoading = false
    }

    @MainActor
    func refresh() async {
        await loadMarkets()
    }
}
```

### View Pattern

```swift
import SwiftUI

struct MarketListView: View {
    @State private var viewModel = MarketListViewModel()

    var body: some View {
        NavigationStack {
            content
                .navigationTitle("Markets")
                .task { await viewModel.loadMarkets() }
                .refreshable { await viewModel.refresh() }
        }
    }

    @ViewBuilder
    private var content: some View {
        if viewModel.isLoading && viewModel.markets.isEmpty {
            LoadingView()
        } else if let error = viewModel.error, viewModel.markets.isEmpty {
            ErrorView(error: error) {
                Task { await viewModel.loadMarkets() }
            }
        } else if viewModel.markets.isEmpty {
            EmptyStateView(
                title: "No Markets",
                message: "Check back later for new markets"
            )
        } else {
            marketList
        }
    }

    private var marketList: some View {
        List(viewModel.markets) { market in
            NavigationLink(value: market) {
                MarketRow(market: market)
            }
        }
        .navigationDestination(for: Market.self) { market in
            MarketDetailView(market: market)
        }
    }
}
```

### Service Pattern

```swift
protocol MarketServiceProtocol {
    func fetchMarkets() async throws -> [Market]
    func getMarket(id: String) async throws -> Market
    func createMarket(_ request: CreateMarketRequest) async throws -> Market
}

final class MarketService: MarketServiceProtocol {
    private let apiClient: APIClientProtocol

    init(apiClient: APIClientProtocol = APIClient.shared) {
        self.apiClient = apiClient
    }

    func fetchMarkets() async throws -> [Market] {
        try await apiClient.get(endpoint: .markets)
    }

    func getMarket(id: String) async throws -> Market {
        try await apiClient.get(endpoint: .market(id: id))
    }

    func createMarket(_ request: CreateMarketRequest) async throws -> Market {
        try await apiClient.post(endpoint: .markets, body: request)
    }
}
```

### API Client Pattern

```swift
protocol APIClientProtocol {
    func get<T: Decodable>(endpoint: Endpoint) async throws -> T
    func post<T: Decodable, U: Encodable>(endpoint: Endpoint, body: U) async throws -> T
}

final class APIClient: APIClientProtocol {
    static let shared = APIClient()

    private let session: URLSession
    private let decoder: JSONDecoder
    private let encoder: JSONEncoder

    init(session: URLSession = .shared) {
        self.session = session
        self.decoder = JSONDecoder()
        self.decoder.dateDecodingStrategy = .iso8601
        self.encoder = JSONEncoder()
        self.encoder.dateEncodingStrategy = .iso8601
    }

    func get<T: Decodable>(endpoint: Endpoint) async throws -> T {
        let request = try endpoint.urlRequest(method: "GET")
        return try await perform(request)
    }

    func post<T: Decodable, U: Encodable>(endpoint: Endpoint, body: U) async throws -> T {
        var request = try endpoint.urlRequest(method: "POST")
        request.httpBody = try encoder.encode(body)
        return try await perform(request)
    }

    private func perform<T: Decodable>(_ request: URLRequest) async throws -> T {
        let (data, response) = try await session.data(for: request)

        guard let httpResponse = response as? HTTPURLResponse else {
            throw APIError.invalidResponse
        }

        guard (200...299).contains(httpResponse.statusCode) else {
            throw APIError.serverError(httpResponse.statusCode)
        }

        return try decoder.decode(T.self, from: data)
    }
}
```

---

## Testing Requirements

### Unit Test Structure

```swift
import XCTest
@testable import ProjectName

final class MarketListViewModelTests: XCTestCase {
    var sut: MarketListViewModel!
    var mockService: MockMarketService!

    override func setUp() {
        super.setUp()
        mockService = MockMarketService()
        sut = MarketListViewModel(marketService: mockService)
    }

    override func tearDown() {
        sut = nil
        mockService = nil
        super.tearDown()
    }

    func test_loadMarkets_success() async {
        // Given
        mockService.marketsToReturn = [.preview]

        // When
        await sut.loadMarkets()

        // Then
        XCTAssertFalse(sut.isLoading)
        XCTAssertNil(sut.error)
        XCTAssertEqual(sut.markets.count, 1)
    }

    func test_loadMarkets_failure() async {
        // Given
        mockService.errorToThrow = APIError.noConnection

        // When
        await sut.loadMarkets()

        // Then
        XCTAssertFalse(sut.isLoading)
        XCTAssertNotNil(sut.error)
        XCTAssertTrue(sut.markets.isEmpty)
    }
}
```

### UI Test Structure

```swift
import XCTest

final class MarketFlowUITests: XCTestCase {
    var app: XCUIApplication!

    override func setUp() {
        super.setUp()
        continueAfterFailure = false
        app = XCUIApplication()
        app.launchArguments = ["--uitesting"]
        app.launch()
    }

    func test_marketListToDetail() {
        // Navigate to markets
        app.tabBars.buttons["Markets"].tap()

        // Wait for list to load
        let firstMarket = app.cells.firstMatch
        XCTAssertTrue(firstMarket.waitForExistence(timeout: 5))

        // Tap to navigate
        firstMarket.tap()

        // Verify detail view
        XCTAssertTrue(app.navigationBars["Market Details"].exists)
    }
}
```

### Coverage Requirements

- **Minimum**: 80% line coverage
- **ViewModels**: 90%+ coverage
- **Services**: 85%+ coverage
- **UI Tests**: Critical flows covered

---

## Deployment Workflow

### Pre-Deployment Checklist

- [ ] All tests passing
- [ ] No compiler warnings
- [ ] Version and build number updated
- [ ] Release notes prepared
- [ ] Screenshots updated (if needed)
- [ ] Certificates and profiles valid

### Build Commands

```bash
# Clean build
xcodebuild clean -scheme ProjectName

# Build for testing
xcodebuild build-for-testing \
  -scheme ProjectName \
  -destination 'platform=iOS Simulator,name=iPhone 15'

# Run tests
xcodebuild test \
  -scheme ProjectName \
  -destination 'platform=iOS Simulator,name=iPhone 15'

# Archive for distribution
xcodebuild archive \
  -scheme ProjectName \
  -archivePath build/ProjectName.xcarchive \
  -configuration Release
```

### Environment Configuration

```swift
enum Environment {
    case development
    case staging
    case production

    var baseURL: URL {
        switch self {
        case .development:
            return URL(string: "https://dev-api.example.com")!
        case .staging:
            return URL(string: "https://staging-api.example.com")!
        case .production:
            return URL(string: "https://api.example.com")!
        }
    }

    static var current: Environment {
        #if DEBUG
        return .development
        #elseif STAGING
        return .staging
        #else
        return .production
        #endif
    }
}
```

---

## Critical Rules

1. **No force unwrapping** in production code
2. **All async code** on MainActor for UI updates
3. **Protocols for dependencies** to enable testing
4. **No hardcoded strings** - use Localizable.strings
5. **Accessibility identifiers** on all interactive elements
6. **80%+ test coverage** minimum
7. **No print statements** in production
8. **Keychain for sensitive data** - never UserDefaults

---

## Related Skills

- `swift-coding-standards.md` - General Swift best practices
- `swiftui-patterns.md` - SwiftUI component patterns
- `ios-security-review/` - Security checklist
- `swift-tdd-workflow/` - Test-driven development
