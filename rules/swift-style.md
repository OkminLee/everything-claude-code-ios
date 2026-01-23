# Swift Coding Style

> Coding style guide for Swift/iOS projects

---

## Value Types & Immutability (CRITICAL)

Swift supports immutability at the language level.

### Prefer struct over class

```swift
// ✅ CORRECT: Value type (Copy-on-Write)
struct User {
    let id: UUID
    let name: String
    let email: String
}

// ❌ AVOID: Reference type (side effects possible)
class MutableUser {
    var name: String  // Can be modified externally
}
```

### Prefer let over var

```swift
// ✅ CORRECT: Immutable by default
let user = User(id: UUID(), name: "John", email: "john@example.com")

// ❌ WRONG: Unnecessary mutability
var user = User(...)  // Use let if not mutating
```

### Copy Instead of Mutate Pattern

```swift
// ✅ CORRECT: Functional update
extension User {
    func withName(_ name: String) -> User {
        User(id: id, name: name, email: email)
    }

    func withEmail(_ email: String) -> User {
        User(id: id, name: name, email: email)
    }
}

let updatedUser = user.withName("Jane")

// ❌ WRONG: Direct mutation
user.name = "Jane"  // Compile error if struct is let
```

---

## File Organization

### Size Guidelines

| File Type | Preferred | Maximum | Action if Exceeded |
|-----------|-----------|---------|-------------------|
| SwiftUI View | 100-200 lines | 300 | Extract to subviews |
| ViewModel | 200-300 lines | 500 | Extract to use cases |
| ViewController | 200-400 lines | 600 | Extract to child VCs |
| Model | 50-100 lines | 200 | Split into components |
| Extension | 50-100 lines | 150 | Split by functionality |

### Feature-Based Structure (Recommended)

```
Features/
├── Login/
│   ├── LoginView.swift           # Main view
│   ├── LoginViewModel.swift      # State management
│   ├── LoginUseCase.swift        # Business logic
│   ├── LoginModel.swift          # Data models
│   └── Components/               # Reusable subviews
│       ├── LoginHeader.swift
│       └── LoginForm.swift
├── Home/
│   ├── HomeView.swift
│   └── ...
Core/
├── Network/
│   ├── APIClient.swift
│   └── Endpoints.swift
├── Storage/
│   ├── KeychainManager.swift
│   └── UserDefaultsManager.swift
└── Extensions/
    ├── String+Extensions.swift
    └── Date+Extensions.swift
```

### SwiftUI vs UIKit Structure

**SwiftUI (Composition-focused):**
```
Features/Profile/
├── ProfileView.swift           # 100-200 lines
├── ProfileViewModel.swift
├── Components/                 # Reusable subviews
│   ├── ProfileHeader.swift     # 50-100 lines each
│   ├── ProfileStats.swift
│   └── ProfileActions.swift
└── ProfileModel.swift
```

**UIKit (Inheritance + Delegate-focused):**
```
Features/Profile/
├── ProfileViewController.swift # 200-400 lines
├── ProfileView.swift           # Custom UIView
├── ProfileViewModel.swift
├── Cells/                      # Table/Collection cells
│   └── ProfileCell.swift
└── ProfileModel.swift
```

### SPM Modularization

```
MyApp/
├── App/                        # Main target
│   └── MyApp.swift
├── Packages/
│   ├── Core/                   # Shared utilities
│   ├── Domain/                 # Business logic
│   ├── Data/                   # Repository, API
│   └── Feature-Login/          # Feature module
│       └── Sources/FeatureLogin/
│           ├── Public/         # External interface
│           └── Internal/       # Implementation
└── Package.swift
```

---

## Error Handling

### throws + do-catch (Recommended)

```swift
enum NetworkError: Error, LocalizedError {
    case invalidURL
    case noConnection
    case serverError(statusCode: Int)
    case decodingFailed(Error)

    var errorDescription: String? {
        switch self {
        case .invalidURL: return "Invalid URL"
        case .noConnection: return "No internet connection"
        case .serverError(let code): return "Server error: \(code)"
        case .decodingFailed: return "Failed to decode response"
        }
    }
}

func fetchData(from urlString: String) async throws -> Data {
    guard let url = URL(string: urlString) else {
        throw NetworkError.invalidURL
    }

    let (data, response) = try await URLSession.shared.data(from: url)

    guard let httpResponse = response as? HTTPURLResponse else {
        throw NetworkError.serverError(statusCode: 0)
    }

    guard (200...299).contains(httpResponse.statusCode) else {
        throw NetworkError.serverError(statusCode: httpResponse.statusCode)
    }

    return data
}

// Usage
do {
    let data = try await fetchData(from: "https://api.example.com")
    // Process data
} catch let error as NetworkError {
    // Handle specific error
    logger.error("Network error: \(error.localizedDescription)")
} catch {
    // Handle unknown error
    logger.error("Unknown error: \(error)")
}
```

### Result Type (For Callbacks)

```swift
func fetchData(completion: @escaping (Result<Data, NetworkError>) -> Void) {
    // ...
    completion(.success(data))
    // or
    completion(.failure(.noConnection))
}
```

### Optional (For Simple Absence)

```swift
// When error details aren't needed
func findUser(by id: UUID) -> User? {
    users.first { $0.id == id }
}
```

---

## Input Validation

### Codable with Validation

```swift
struct UserInput: Codable {
    let email: String
    let age: Int

    enum CodingKeys: String, CodingKey {
        case email, age
    }

    init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)

        email = try container.decode(String.self, forKey: .email)
        guard email.contains("@") && email.contains(".") else {
            throw DecodingError.dataCorrupted(
                .init(codingPath: [CodingKeys.email],
                      debugDescription: "Invalid email format")
            )
        }

        age = try container.decode(Int.self, forKey: .age)
        guard (0...150).contains(age) else {
            throw DecodingError.dataCorrupted(
                .init(codingPath: [CodingKeys.age],
                      debugDescription: "Age must be 0-150")
            )
        }
    }
}
```

### Property Wrapper Validation

```swift
@propertyWrapper
struct Clamped<Value: Comparable> {
    private var value: Value
    private let range: ClosedRange<Value>

    var wrappedValue: Value {
        get { value }
        set { value = min(max(range.lowerBound, newValue), range.upperBound) }
    }

    init(wrappedValue: Value, _ range: ClosedRange<Value>) {
        self.range = range
        self.value = min(max(range.lowerBound, wrappedValue), range.upperBound)
    }
}

struct Player {
    @Clamped(0...100) var health: Int = 100
    @Clamped(0...999) var score: Int = 0
}
```

### Validated Types (Compile-time Safety)

```swift
struct Email: RawRepresentable, Codable, Sendable {
    let rawValue: String

    init?(rawValue: String) {
        guard rawValue.contains("@"),
              rawValue.contains("."),
              rawValue.count >= 5 else {
            return nil
        }
        self.rawValue = rawValue
    }
}

// Usage: Type system prevents invalid emails
func sendEmail(to email: Email) { ... }
```

### Property Wrapper Guidelines

```swift
// ✅ Appropriate: Apple-provided Wrappers
@State, @Binding, @StateObject, @ObservedObject
@Published, @AppStorage, @SceneStorage
@Environment, @EnvironmentObject

// ✅ Appropriate: Abstracting repeated patterns
@Clamped(0...100) var health: Int
@UserDefault("key") var setting: Bool
@Injected var service: APIService

// ⚠️ Caution: No more than 3 stacked wrappers
@Trimmed @Lowercased @Validated var email: String  // Hurts readability

// ❌ Avoid: Over-engineering
@MyWrapper var x: Int  // Simple logic wrapped unnecessarily
```

---

## Logging

### os_log / Logger (iOS 14+)

```swift
import os

private let logger = Logger(subsystem: "com.company.app", category: "Network")

// Log levels
logger.debug("Request started")           // Development only
logger.info("User logged in")             // General information
logger.notice("Cache cleared")            // Notable events
logger.warning("Rate limit approaching")  // Potential issues
logger.error("Request failed: \(error.localizedDescription, privacy: .public)")
logger.fault("Critical system error")     // Severe errors

// Privacy: Mask sensitive data
logger.info("User: \(user.email, privacy: .private)")  // Shows <private> in release
logger.info("ID: \(user.id, privacy: .public)")        // Always visible
```

### print() is Forbidden

```swift
// ❌ NEVER in production
print("Debug: \(value)")
debugPrint(object)

// ✅ Use Logger
logger.debug("Debug: \(value)")

// ✅ Or use preprocessor for development only
#if DEBUG
print("Development only message")
#endif
```

---

## Swift 6 Concurrency Style

### @MainActor Explicit Declaration

```swift
// ✅ CORRECT: Explicit @MainActor
@MainActor
final class ProfileViewModel: ObservableObject {
    @Published private(set) var profile: Profile?

    func loadProfile() async {
        profile = await profileService.fetch()
    }
}

// ❌ WRONG: Implicit, unclear thread safety
class ProfileViewModel: ObservableObject {
    var profile: Profile?

    func loadProfile() {
        Task {
            profile = await profileService.fetch()  // Swift 6 error
        }
    }
}
```

### Sendable Compliance

```swift
// ✅ CORRECT: Sendable-safe types
struct UserData: Sendable {
    let id: UUID
    let name: String
}

// ✅ CORRECT: @Sendable closure
Task { @Sendable in
    await performWork()
}

// ❌ WRONG: Non-Sendable captured
let viewModel = ViewModel()  // Non-Sendable class
Task {
    await viewModel.load()  // Capture warning in Swift 6
}
```

### Task Cancellation

```swift
@MainActor
final class SearchViewModel: ObservableObject {
    private var searchTask: Task<Void, Never>?

    func search(query: String) {
        // Cancel previous search
        searchTask?.cancel()

        searchTask = Task {
            // Check cancellation
            guard !Task.isCancelled else { return }

            let results = try? await searchService.search(query)

            // Check again after async work
            guard !Task.isCancelled else { return }

            self.results = results ?? []
        }
    }

    deinit {
        searchTask?.cancel()
    }
}
```

### Observation (iOS 17+)

```swift
// ✅ Modern approach
@Observable
@MainActor
final class ProfileViewModel {
    private(set) var profile: Profile?
    @ObservationIgnored private var cancellables = Set<AnyCancellable>()

    func loadProfile() async {
        profile = await profileService.fetch()
    }
}

// In SwiftUI View
struct ProfileView: View {
    @Bindable var viewModel: ProfileViewModel

    var body: some View {
        // Automatically tracks changes
        Text(viewModel.profile?.name ?? "Loading...")
    }
}
```

---

## SwiftUI Best Practices

### Extract ViewBuilder

```swift
// ❌ WRONG: Deep nesting
var body: some View {
    VStack {
        HStack {
            VStack {
                Text("Title")
                Text("Subtitle")
            }
            Spacer()
            Button("Action") { }
        }
        // ... more nested views
    }
}

// ✅ CORRECT: Extract subviews
var body: some View {
    VStack {
        headerView
        contentView
        footerView
    }
}

@ViewBuilder
private var headerView: some View {
    HStack {
        titleStack
        Spacer()
        actionButton
    }
}

@ViewBuilder
private var titleStack: some View {
    VStack(alignment: .leading) {
        Text("Title")
        Text("Subtitle")
    }
}
```

### Custom Modifier for Reuse

```swift
// ✅ Extract repeated modifier chains
struct CardStyle: ViewModifier {
    func body(content: Content) -> some View {
        content
            .padding()
            .background(Color(.systemBackground))
            .cornerRadius(12)
            .shadow(radius: 4)
    }
}

extension View {
    func cardStyle() -> some View {
        modifier(CardStyle())
    }
}

// Usage
Text("Content")
    .cardStyle()
```

### Preview Macro (Xcode 15+)

```swift
// ✅ Modern preview syntax
#Preview {
    ProfileView(viewModel: .preview)
}

#Preview("Dark Mode") {
    ProfileView(viewModel: .preview)
        .preferredColorScheme(.dark)
}

// ❌ Legacy syntax
struct ProfileView_Previews: PreviewProvider {
    static var previews: some View {
        ProfileView(viewModel: .preview)
    }
}
```

---

## Code Quality Checklist

### Structure & Naming
- [ ] Follows Swift API Design Guidelines
- [ ] Functions < 40 lines (max 80)
- [ ] SwiftUI View < 200 lines, ViewController < 400 lines
- [ ] Total file < 500 lines (max 800)
- [ ] Nesting < 3 levels
- [ ] 1 major type per file

### Memory & Performance
- [ ] Prevent retain cycles (`[weak self]` in closures)
- [ ] `@MainActor` for UI updates
- [ ] Heavy operations off main thread
- [ ] `lazy` for expensive computed properties
- [ ] No force unwrapping (except `@IBOutlet`)

### Swift 6 Concurrency
- [ ] Explicit `@MainActor` declaration
- [ ] Verify `Sendable` conformance
- [ ] Handle `Task` cancellation
- [ ] Use `nonisolated` explicitly when needed

### Swift Best Practices
- [ ] `struct` > `class` (value types)
- [ ] `let` > `var` (immutability)
- [ ] `guard` for early returns
- [ ] `private` by default
- [ ] No unused imports or dead code

### Error Handling
- [ ] Meaningful `Error` types with context
- [ ] All `throws` functions have proper `do-catch`
- [ ] `async/await` with Task cancellation handling

### Logging & Debug
- [ ] No `print()` in production → `os_log`/`Logger`
- [ ] No hardcoded values → constants or config
- [ ] No TODO/FIXME without tracking issue
- [ ] No commented-out code

### Property Wrappers
- [ ] No more than 3 stacked wrappers
- [ ] Document custom wrappers
- [ ] Prefer Apple-provided wrappers

### SwiftUI Specific
- [ ] Extract `@ViewBuilder` to prevent nesting
- [ ] Modifier chains > 5 levels → Custom Modifier
- [ ] Use `#Preview` macro

### iOS Specific
- [ ] Accessibility labels for interactive elements
- [ ] Localization-ready strings
- [ ] Support Dynamic Type
- [ ] Dark mode compatibility

---

## Quick Reference

### Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Type | UpperCamelCase | `UserProfile` |
| Function | lowerCamelCase | `fetchUserProfile()` |
| Variable | lowerCamelCase | `userProfile` |
| Constant | lowerCamelCase | `maxRetryCount` |
| Protocol | UpperCamelCase | `Fetchable`, `UserProviding` |
| Enum case | lowerCamelCase | `.loading`, `.success` |

### Access Control

| Level | Scope | Use Case |
|-------|-------|----------|
| `private` | Same file | Default for implementation |
| `fileprivate` | Same file | Rarely used |
| `internal` | Same module | Default (omit keyword) |
| `public` | Other modules | API surface |
| `open` | Other modules + subclass | Framework extension points |

### Common Patterns

```swift
// Guard early return
guard let value = optionalValue else { return }

// Defer for cleanup
func process() {
    let resource = acquire()
    defer { release(resource) }
    // Use resource
}

// Result builder for DSL
@resultBuilder
struct ArrayBuilder { ... }
```
