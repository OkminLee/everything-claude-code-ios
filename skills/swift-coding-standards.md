---
name: swift-coding-standards
description: Universal coding standards, best practices, and patterns for Swift and iOS development following Apple's API Design Guidelines.
---

# Swift Coding Standards & Best Practices

Universal coding standards applicable across all iOS/Swift projects.

## Code Quality Principles

### 1. Clarity at the Point of Use
- Code should read naturally at call sites
- Prioritize clarity over brevity
- Use descriptive names that explain purpose

### 2. KISS (Keep It Simple, Stupid)
- Simplest solution that works
- Avoid over-engineering
- No premature optimization
- Easy to understand > clever code

### 3. DRY (Don't Repeat Yourself)
- Extract common logic into functions
- Create reusable components
- Use protocol extensions for shared behavior
- Avoid copy-paste programming

### 4. YAGNI (You Aren't Gonna Need It)
- Don't build features before they're needed
- Avoid speculative generality
- Add complexity only when required
- Start simple, refactor when needed

## Swift Naming Conventions

### Type Naming (PascalCase)

```swift
// ✅ GOOD: Clear, descriptive type names
struct UserProfile { }
class NetworkManager { }
enum PaymentStatus { }
protocol DataFetching { }

// ❌ BAD: Unclear or abbreviated
struct UP { }
class NM { }
enum PS { }
```

### Variable & Function Naming (camelCase)

```swift
// ✅ GOOD: Descriptive names
let marketSearchQuery = "election"
var isUserAuthenticated = true
let totalRevenue: Decimal = 1000

// ❌ BAD: Unclear names
let q = "election"
var flag = true
let x = 1000
```

### Function Naming

```swift
// ✅ GOOD: Verb-based, reads naturally at call site
func fetchMarketData(for marketId: String) async throws -> Market { }
func calculateSimilarity(between first: [Double], and second: [Double]) -> Double { }
func isValidEmail(_ email: String) -> Bool { }

// Usage reads like English
let market = try await fetchMarketData(for: "market-123")
let score = calculateSimilarity(between: vectorA, and: vectorB)
if isValidEmail(input) { }

// ❌ BAD: Unclear or noun-only
func market(_ id: String) -> Market { }
func similarity(_ a: [Double], _ b: [Double]) -> Double { }
```

### Boolean Naming

```swift
// ✅ GOOD: Reads as assertion
var isLoading = false
var hasCompletedOnboarding = true
var canSubmitForm = false
var shouldShowAlert = true

// ❌ BAD: Unclear intent
var loading = false
var onboarding = true
var submit = false
```

## Immutability Pattern (CRITICAL)

```swift
// ✅ ALWAYS prefer let over var
let user = User(name: "John")
let items = [1, 2, 3]

// ✅ Create new instances instead of mutating
let updatedUser = User(
    id: user.id,
    name: "New Name",
    email: user.email
)

let updatedItems = items + [4]

// ❌ NEVER mutate when avoidable
var user = User(name: "John")
user.name = "New Name"  // Avoid mutation

var items = [1, 2, 3]
items.append(4)  // Avoid in-place mutation
```

## Error Handling

### Using Result Type

```swift
// ✅ GOOD: Comprehensive error handling
enum NetworkError: Error {
    case invalidURL
    case noData
    case decodingFailed(Error)
    case serverError(Int)
}

func fetchData(from urlString: String) async -> Result<Data, NetworkError> {
    guard let url = URL(string: urlString) else {
        return .failure(.invalidURL)
    }

    do {
        let (data, response) = try await URLSession.shared.data(from: url)

        guard let httpResponse = response as? HTTPURLResponse else {
            return .failure(.noData)
        }

        guard (200...299).contains(httpResponse.statusCode) else {
            return .failure(.serverError(httpResponse.statusCode))
        }

        return .success(data)
    } catch {
        return .failure(.noData)
    }
}

// ❌ BAD: No error handling
func fetchData(from url: String) async -> Data? {
    let data = try? await URLSession.shared.data(from: URL(string: url)!)
    return data?.0
}
```

### Using Throws

```swift
// ✅ GOOD: Typed throws (Swift 6+)
func parseJSON<T: Decodable>(_ data: Data) throws(DecodingError) -> T {
    try JSONDecoder().decode(T.self, from: data)
}

// ✅ GOOD: async/await with error handling
func loadMarkets() async throws -> [Market] {
    let result = await fetchData(from: API.marketsURL)

    switch result {
    case .success(let data):
        return try parseJSON(data)
    case .failure(let error):
        throw error
    }
}
```

## Async/Await Best Practices

```swift
// ✅ GOOD: Parallel execution with TaskGroup
func loadDashboard() async throws -> Dashboard {
    async let users = fetchUsers()
    async let markets = fetchMarkets()
    async let stats = fetchStats()

    return try await Dashboard(
        users: users,
        markets: markets,
        stats: stats
    )
}

// ✅ GOOD: Structured concurrency
func processItems(_ items: [Item]) async throws -> [ProcessedItem] {
    try await withThrowingTaskGroup(of: ProcessedItem.self) { group in
        for item in items {
            group.addTask {
                try await process(item)
            }
        }

        var results: [ProcessedItem] = []
        for try await result in group {
            results.append(result)
        }
        return results
    }
}

// ❌ BAD: Sequential when unnecessary
func loadDashboard() async throws -> Dashboard {
    let users = try await fetchUsers()
    let markets = try await fetchMarkets()  // Waits for users
    let stats = try await fetchStats()       // Waits for markets

    return Dashboard(users: users, markets: markets, stats: stats)
}
```

## Type Safety

```swift
// ✅ GOOD: Strong typing with custom types
struct MarketID: Hashable {
    let rawValue: String
}

struct Market {
    let id: MarketID
    let name: String
    let status: MarketStatus
    let createdAt: Date
}

enum MarketStatus: String, Codable {
    case active
    case resolved
    case closed
}

func getMarket(id: MarketID) async throws -> Market {
    // Implementation
}

// ❌ BAD: Using raw types everywhere
func getMarket(id: String) async throws -> [String: Any] {
    // No type safety
}
```

## SwiftUI Best Practices

### View Structure

```swift
// ✅ GOOD: Small, focused views
struct MarketCard: View {
    let market: Market

    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            Text(market.name)
                .font(.headline)

            StatusBadge(status: market.status)

            MarketMetrics(market: market)
        }
        .padding()
        .background(Color(.systemBackground))
        .cornerRadius(12)
    }
}

// ❌ BAD: Massive view with all logic
struct MarketCard: View {
    let market: Market
    @State private var isLoading = false
    @State private var error: Error?
    // 100+ lines in body...
}
```

### State Management

```swift
// ✅ GOOD: Appropriate property wrappers
struct ContentView: View {
    @State private var count = 0                    // Local state
    @Binding var isPresented: Bool                  // Parent-owned state
    @Environment(\.dismiss) private var dismiss    // Environment value
    @StateObject private var viewModel = ViewModel() // Observable object

    var body: some View {
        Button("Increment: \(count)") {
            count += 1
        }
    }
}

// ❌ BAD: Wrong property wrapper
struct ContentView: View {
    @ObservedObject var viewModel = ViewModel()  // Creates new instance each render!
}
```

### View Modifiers

```swift
// ✅ GOOD: Custom view modifiers for reusable styling
struct CardStyle: ViewModifier {
    func body(content: Content) -> some View {
        content
            .padding()
            .background(Color(.systemBackground))
            .cornerRadius(12)
            .shadow(radius: 2)
    }
}

extension View {
    func cardStyle() -> some View {
        modifier(CardStyle())
    }
}

// Usage
MarketCard(market: market)
    .cardStyle()
```

## Protocol-Oriented Design

```swift
// ✅ GOOD: Protocol with default implementation
protocol Loadable {
    associatedtype Output
    var isLoading: Bool { get }
    var error: Error? { get }
    func load() async throws -> Output
}

extension Loadable {
    var hasError: Bool { error != nil }
}

// ✅ GOOD: Protocol composition
typealias DataSource = Loadable & Identifiable & Hashable
```

## API Design Standards

### Response Format

```swift
// ✅ GOOD: Consistent response structure
struct APIResponse<T: Decodable>: Decodable {
    let success: Bool
    let data: T?
    let error: String?
    let meta: ResponseMeta?
}

struct ResponseMeta: Decodable {
    let total: Int
    let page: Int
    let limit: Int
}

// Usage
func fetchMarkets() async throws -> APIResponse<[Market]> {
    let data = try await networkClient.get("/api/markets")
    return try JSONDecoder().decode(APIResponse<[Market]>.self, from: data)
}
```

### Input Validation

```swift
// ✅ GOOD: Validation before processing
struct CreateMarketRequest {
    let name: String
    let description: String
    let endDate: Date

    enum ValidationError: Error {
        case nameTooShort
        case nameTooLong
        case descriptionEmpty
        case endDateInPast
    }

    func validate() throws {
        guard name.count >= 1 else { throw ValidationError.nameTooShort }
        guard name.count <= 200 else { throw ValidationError.nameTooLong }
        guard !description.isEmpty else { throw ValidationError.descriptionEmpty }
        guard endDate > Date() else { throw ValidationError.endDateInPast }
    }
}

// Usage
func createMarket(_ request: CreateMarketRequest) async throws -> Market {
    try request.validate()
    return try await api.post("/markets", body: request)
}
```

## File Organization

### Project Structure

```
ProjectName/
├── App/
│   ├── ProjectNameApp.swift
│   └── AppDelegate.swift
├── Features/
│   ├── Markets/
│   │   ├── Views/
│   │   ├── ViewModels/
│   │   └── Models/
│   └── Auth/
│       ├── Views/
│       ├── ViewModels/
│       └── Models/
├── Core/
│   ├── Network/
│   ├── Storage/
│   └── Extensions/
├── UI/
│   ├── Components/
│   └── Styles/
└── Resources/
    ├── Assets.xcassets
    └── Localizable.strings
```

### File Naming

```
Views/MarketListView.swift         # Views suffix
ViewModels/MarketListViewModel.swift  # ViewModel suffix
Models/Market.swift                # Model (no suffix)
Services/MarketService.swift       # Service suffix
Extensions/Date+Formatting.swift   # Type+Category
```

## Comments & Documentation

### When to Comment

```swift
// ✅ GOOD: Explain WHY, not WHAT
// Use exponential backoff to avoid overwhelming the API during outages
let delay = min(pow(2.0, Double(retryCount)) * 1000, 30000)

// Deliberately using mutation here for performance with large arrays
items.append(contentsOf: newItems)

// ❌ BAD: Stating the obvious
// Increment counter by 1
count += 1

// Set name to user's name
name = user.name
```

### Documentation Comments

```swift
/// Searches markets using semantic similarity.
///
/// - Parameters:
///   - query: Natural language search query
///   - limit: Maximum number of results (default: 10)
/// - Returns: Array of markets sorted by similarity score
/// - Throws: `NetworkError` if API fails, `DecodingError` if response invalid
///
/// Example:
/// ```swift
/// let results = try await searchMarkets(query: "election", limit: 5)
/// print(results.first?.name) // "Trump vs Biden"
/// ```
func searchMarkets(query: String, limit: Int = 10) async throws -> [Market] {
    // Implementation
}
```

## Performance Best Practices

### Lazy Properties

```swift
// ✅ GOOD: Lazy initialization for expensive objects
class ViewModel {
    lazy var dateFormatter: DateFormatter = {
        let formatter = DateFormatter()
        formatter.dateStyle = .medium
        formatter.timeStyle = .short
        return formatter
    }()
}
```

### Collection Operations

```swift
// ✅ GOOD: Lazy evaluation for large collections
let processedItems = items.lazy
    .filter { $0.isActive }
    .map { $0.transformed }
    .prefix(10)

// ✅ GOOD: Use appropriate collection types
let uniqueIds = Set(items.map(\.id))  // O(1) lookup
let idToItem = Dictionary(uniqueKeysWithValues: items.map { ($0.id, $0) })
```

### Avoid Retain Cycles

```swift
// ✅ GOOD: Weak self in closures
class ViewModel: ObservableObject {
    func loadData() {
        Task { [weak self] in
            guard let self else { return }
            let data = try await fetchData()
            await MainActor.run {
                self.items = data
            }
        }
    }
}

// ❌ BAD: Strong reference cycle
class ViewModel: ObservableObject {
    func loadData() {
        Task {
            let data = try await fetchData()
            self.items = data  // Captures self strongly
        }
    }
}
```

## Testing Standards

### Test Structure (Given-When-Then)

```swift
func test_searchMarkets_returnsRelevantResults() async throws {
    // Given
    let query = "election"
    let sut = MarketSearchService()

    // When
    let results = try await sut.search(query: query, limit: 10)

    // Then
    XCTAssertFalse(results.isEmpty)
    XCTAssertTrue(results.first?.name.lowercased().contains("election") ?? false)
}
```

### Test Naming

```swift
// ✅ GOOD: Descriptive test names
func test_fetchMarkets_whenNetworkFails_throwsNetworkError() { }
func test_createMarket_withInvalidName_throwsValidationError() { }
func test_searchMarkets_returnsEmptyArray_whenNoMatches() { }

// ❌ BAD: Vague test names
func testSearch() { }
func testError() { }
```

## Code Smell Detection

Watch for these anti-patterns:

### 1. Massive View Controllers/ViewModels
```swift
// ❌ BAD: 500+ lines
class ViewController: UIViewController {
    // Too much responsibility
}

// ✅ GOOD: Split into focused components
class MarketListViewController: UIViewController { }
class MarketListViewModel { }
class MarketService { }
```

### 2. Force Unwrapping
```swift
// ❌ BAD: Crashes on nil
let url = URL(string: urlString)!
let value = dictionary["key"]!

// ✅ GOOD: Safe unwrapping
guard let url = URL(string: urlString) else {
    throw ValidationError.invalidURL
}

if let value = dictionary["key"] {
    // Use value
}
```

### 3. Magic Numbers
```swift
// ❌ BAD: Unexplained numbers
if retryCount > 3 { }
DispatchQueue.main.asyncAfter(deadline: .now() + 0.5) { }

// ✅ GOOD: Named constants
private enum Constants {
    static let maxRetries = 3
    static let debounceDelay: TimeInterval = 0.5
}

if retryCount > Constants.maxRetries { }
```

**Remember**: Code quality is not negotiable. Clear, maintainable code enables rapid development and confident refactoring.
