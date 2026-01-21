# Phase 1.1: Agents Learning Notes

> Date: 2026-01-21
> Last Updated: Senior iOS Engineer 2nd Review Feedback Applied

---

## 1. planner.md Analysis

### File Structure
```yaml
name: planner
description: Expert planning specialist for complex features and refactoring
tools: Read, Grep, Glob  # Read-only
model: opus
```

### Key Patterns
1. **Frontmatter**: 4 fields - name, description, tools, model
2. **Read-only Principle**: Planning agent only analyzes, doesn't modify code
3. **4-Step Process**: Requirements Analysis → Architecture Review → Step Breakdown → Implementation Order
4. **Plan Format**: Overview, Requirements, Architecture Changes, Implementation Steps, Testing Strategy, Risks, Success Criteria

### iOS Adaptation

#### Applicability Assessment
| Item | Verdict | Notes |
|------|---------|-------|
| Frontmatter structure | ✅ | Use as-is |
| 4-step process | ✅ | Use as-is |
| Plan format | 🔄 | .ts → .swift |
| Red Flags | 🔄 | Add iOS-specific items |
| Testing Strategy | 🔄 | Change to XCTest/XCUITest |

---

### Content to Add for iOS Version

#### 1. iOS Red Flags Section (2nd Review Feedback Applied)

```markdown
## iOS Red Flags to Check

### Basic Checks
- Main thread blocking (heavy work on main)
- Retain cycles (strong reference cycles)
- Missing @MainActor for UI updates
- Force unwrapping (!!) without safety
- Large view body (>30 lines in SwiftUI)
- Missing Accessibility labels
- Hardcoded strings (not localized)
- Missing error handling for async/await
- Memory leaks (unreleased resources)
- Missing weak/unowned in closures

### Concurrency
- Task cancellation not handled (Task continues after cancellation)
- Actor isolation violation (nonisolated overuse)
- Data race potential (Sendable non-compliance)

### SwiftUI Specific
- @State declared in parent View (should be managed by child)
- ObservableObject overuse (prefer @Observable on iOS 17+)
- Business logic inside View

### Memory
- NotificationCenter observer not removed
- Timer not invalidated
- URLSession task not cancelled

### API/Network
- Network error handling missing (offline state not handled)
- API response timeout not configured
- Infinite loading state (stuck when error occurs during loading)
- One-shot requests without retry logic

### Security
- Sensitive data stored in UserDefaults (should use Keychain)
- Hardcoded API keys/secrets
- Certificate pinning not applied (when required)
- Sensitive information logged

### Testing
- Untestable singletons
- Hardcoded dependencies (DI not applied)
```

#### 2. App Store Guideline Section (2024+ Updates Included)

```markdown
## App Store Guideline Check

### Required Checks (High Rejection Rate)
- [ ] App explorable without login? (Guest mode)
- [ ] Sign in with Apple supported? (Required if social login exists)
- [ ] Privacy policy URL valid?
- [ ] No in-app purchase bypass? (No web payment redirection)

### UI/UX Checks
- [ ] System back gesture works?
- [ ] Safe Area respected?
- [ ] Dynamic Type supported?
- [ ] Dark Mode supported?

### Technical Checks
- [ ] Background mode used appropriately? (No unnecessary background execution)
- [ ] Location permission purpose clearly stated?
- [ ] ATT (App Tracking Transparency) implemented? (When using IDFA)

### Accessibility Checks
- [ ] VoiceOver supported
- [ ] Dynamic Type supported
- [ ] Sufficient color contrast
- [ ] Touch targets minimum 44x44pt

### 2024+ New Checks
- [ ] EU region: Alternative payment system considered? (DMA regulation)
- [ ] StoreKit External Purchase Link applied for external links?
- [ ] Data deletion option provided on app deletion? (Account deletion mandatory)
- [ ] Privacy Manifest (PrivacyInfo.xcprivacy) created?
- [ ] Required Reason API usage purpose specified?
```

#### 3. Clean Architecture Layer Section (Practical Perspective)

```markdown
## Architecture Changes by Layer

### Option A: Clean Architecture (MVVM)

#### Domain Layer
- [ ] Entities: [changes]
- [ ] Use Cases: [changes]
- [ ] Repository Protocols: [changes]
- [ ] Error Types: [changes]

#### Data Layer
- [ ] Repository Implementations: [changes]
- [ ] Data Sources (Remote/Local): [changes]
- [ ] DTOs/Mappers: [changes]
- [ ] API Endpoints: [changes]

#### Presentation Layer
- [ ] ViewModels: [changes]
- [ ] Views: [changes]
- [ ] Coordinators/Routers: [changes]
- [ ] UI State: [changes]

#### DI/Infrastructure
- [ ] DI Container configuration: [changes]
- [ ] Environment configuration: [changes]

### Option B: TCA (The Composable Architecture)

- [ ] State: [changes]
- [ ] Action: [changes]
- [ ] Reducer: [changes]
- [ ] Effect/Dependencies: [changes]
- [ ] View: [changes]
```

#### 4. iOS Testing Strategy Section (Practical Perspective)

```markdown
## iOS Testing Strategy

### Unit Tests (XCTest)
- [ ] ViewModel/Reducer logic tests
- [ ] UseCase tests
- [ ] Repository tests (with Mocks)
- [ ] Mapper/DTO conversion tests
- [ ] Error case tests
- [ ] Edge case tests (empty array, nil, etc.)

### UI Tests (XCUITest)
- [ ] Core user flow tests
- [ ] Accessibility identifier verification
- [ ] Error state UI tests

### Snapshot Tests (Optional)
- [ ] UI component snapshots
- [ ] Dark mode/Light mode snapshots
- [ ] Various device size snapshots

### Integration Tests
- [ ] API integration tests (with Mock Server)
- [ ] Database integration tests

### Performance Tests
- [ ] Large data processing performance
- [ ] Memory usage measurement
- [ ] App launch time measurement
```

---

### Conclusion
- `planner.md` can be used almost as-is for iOS
- iOS-specific Red Flags added (Concurrency, SwiftUI, Memory, API/Network, Security, Testing)
- App Store Guidelines added (actual rejection reasons + 2024+ new regulations)
- Both Clean Architecture + TCA options supported (DI/Infrastructure layer added)
- iOS Testing Strategy section added (including Performance Tests)

---

## 2. architect.md Analysis

### File Structure
```yaml
name: architect
description: Software architecture specialist for system design, scalability, and technical decision-making
tools: Read, Grep, Glob  # Read-only
model: opus
```

### Key Patterns
1. **4-Step Process**: Current State Analysis → Requirements Gathering → Design Proposal → Trade-off Analysis
2. **Architectural Principles**: Modularity, Scalability, Maintainability, Security, Performance
3. **ADR (Architecture Decision Records)**: Important architecture decision documentation template
4. **System Design Checklist**: Functional/Non-functional requirements, Technical design, Operations checklist
5. **Red Flags**: Architecture anti-pattern list

### iOS Adaptation Analysis

| Item | Applicability | Notes |
|------|--------------|-------|
| Frontmatter | ✅ As-is | |
| 4-step process | ✅ As-is | |
| Architectural Principles | ✅ As-is | SOLID, modularity, etc. |
| Common Patterns | 🔄 Full revision | Replace with iOS patterns |
| ADR template | ✅ As-is | |
| System Design Checklist | 🔄 Partial revision | Operations → App Store deployment |
| Red Flags | ✅ As-is | |
| Project Example | 🔄 Full revision | Replace with iOS stack |

### Content to Modify for iOS Version

#### Common Patterns → Replace with iOS Patterns (2nd Review Feedback Applied)

```markdown
### iOS Presentation Patterns
- **MVVM**: ViewModel manages View state
- **TCA**: State, Action, Reducer, Effect structure
- **Coordinator/Router**: Screen navigation logic separation
- **View Composition**: Build large screens from small View combinations

### iOS Data Patterns
- **Repository Pattern**: Data source abstraction
- **UseCase Pattern**: Business logic encapsulation
- **DTO/Entity Mapping**: Network model ↔ Domain model

### iOS Concurrency Patterns
- **async/await**: Asynchronous operations
- **Actor**: State isolation and concurrency safety
- **Combine**: Reactive data streams
- **@MainActor**: UI update guarantee

### iOS Navigation Patterns
- **NavigationStack**: iOS 16+ declarative navigation
- **NavigationPath**: Programmatic navigation state management
- **Sheet/FullScreenCover**: Modal presentation
- **Deep Link Handling**: URL Scheme / Universal Link handling

### iOS State Management Patterns
- **@State/@Binding**: Local View state
- **@StateObject/@ObservedObject**: Reference type state
- **@Observable (iOS 17+)**: Observation framework
- **@Environment**: Dependency injection
```

#### Project Example → Replace with iOS Example (2nd Review Feedback Applied)

```markdown
### iOS Architecture Example
- **UI Framework**: SwiftUI + UIKit (legacy)
- **Architecture**: MVVM + Clean Architecture or TCA
- **DI**: Swinject, Factory, or manual
- **Networking**: URLSession + async/await
- **Persistence**: CoreData, SwiftData, Realm
- **State**: Combine, @Observable, TCA Store

### iOS App Scale Strategy

#### MVP
- File count: ~50
- Team size: 1-2
- Structure: Single module, basic MVVM

#### Growth
- File count: ~200
- Team size: 3-5
- Structure: Feature modules, Clean Architecture
- Modularization: Feature-based separation begins

#### Enterprise
- File count: 500+
- Team size: 5+
- Structure: Multi-module, independent builds, micro-features
- Modularization: SPM/CocoaPods based independent modules
- Build time optimization: Incremental builds, caching
```

---

## 3. tdd-guide.md Analysis

### File Structure
```yaml
name: tdd-guide
description: Test-Driven Development specialist enforcing write-tests-first methodology
tools: Read, Write, Edit, Bash, Grep  # Can modify code!
model: opus
```

### Key Patterns
1. **TDD Workflow**: RED (failing test) → GREEN (minimal impl) → REFACTOR → Verify coverage
2. **Test Types**: Unit, Integration, E2E
3. **Coverage Target**: 80%+ (Branches, Functions, Lines, Statements)
4. **Edge Cases**: Null, Empty, Invalid, Boundaries, Errors, Race Conditions, Large Data, Special Chars
5. **Test Smells**: Implementation details testing, dependent tests

### iOS Adaptation Analysis

| Item | Applicability | Notes |
|------|--------------|-------|
| Frontmatter | ✅ As-is | |
| TDD Workflow | ✅ As-is | Same RED-GREEN-REFACTOR |
| Test Types | 🔄 Major revision | Jest → Swift Testing / XCTest |
| Code Examples | 🔄 Full rewrite | TypeScript → Swift |
| Mocking | 🔄 Major revision | Jest mocks → Protocol-based mocks |
| Edge Cases | ✅ As-is | Universal concepts |
| Coverage Report | 🔄 Revision | npm → xcodebuild + xcov |

### Content to Modify for iOS Version

#### 1. Test Framework Options (Hybrid Support)

```markdown
### Option A: Swift Testing (iOS 17+, Xcode 15+) - Recommended
- Modern macro-based syntax (`@Test`, `@Suite`, `#expect`)
- Native async/await support
- Parameterized tests built-in
- Parallel execution by default

### Option B: XCTest (iOS 16 and below, Legacy)
- Traditional XCTestCase class-based
- Broader compatibility
- Use for projects requiring older iOS support
```

#### 2. XCTest → Swift Testing Conversion Table

| XCTest | Swift Testing |
|--------|---------------|
| `import XCTest` | `import Testing` |
| `class FooTests: XCTestCase` | `@Suite struct FooTests` |
| `func testXxx()` | `@Test func xxx()` |
| `XCTAssertEqual(a, b)` | `#expect(a == b)` |
| `XCTAssertTrue(x)` | `#expect(x)` |
| `XCTAssertFalse(x)` | `#expect(!x)` |
| `XCTAssertNil(x)` | `#expect(x == nil)` |
| `XCTAssertNotNil(x)` | `#expect(x != nil)` |
| `XCTAssertGreaterThan(a, b)` | `#expect(a > b)` |
| `XCTAssertLessThan(a, b)` | `#expect(a < b)` |
| `XCTAssertThrowsError` | `#expect(throws:)` |
| `XCTUnwrap(optional)` | `try #require(optional)` |
| `XCTFail("message")` | `Issue.record("message")` |
| `XCTSkip("reason")` | `throw TestSkipped("reason")` |
| `setUpWithError()` | `init() async throws` |
| `tearDown()` | `deinit` |
| `addTeardownBlock {}` | `deinit` or `@Test(.serialized)` |
| `measure {}` | Not available (use XCTest for performance tests) |
| `expectation/fulfill` | `confirmation {}` |

**Important: `#require` for Optional Unwrapping**
```swift
// Unwrap optional with early test failure if nil
@Test func user_hasValidEmail() throws {
    let user = try #require(fetchUser())  // Fails test if nil
    #expect(user.email.contains("@"))
}
```

#### 3. TDD Workflow → Swift Testing

```swift
// Step 1: Write failing test (RED)
import Testing

@Suite struct MarketSearchTests {
    let sut: MarketSearchUseCase

    init() {
        sut = MarketSearchUseCase(repository: MockMarketRepository())
    }

    @Test func search_returnsSemanticallySimarMarkets() async throws {
        // When
        let results = try await sut.search(query: "election")

        // Then
        #expect(results.count == 5)
        #expect(results[0].name.contains("Trump"))
    }
}
```

**Important: `@Suite` init() Behavior**
```swift
// ⚠️ init() is called before EACH @Test method
@Suite struct DatabaseTests {
    let database: Database

    init() async throws {
        // Called for every single @Test - avoid heavy setup here
        database = try await Database.inMemory()
    }

    // For shared expensive resources, use static or external setup
}
```

#### 4. Swift Testing Advanced Features

```swift
// Parameterized Tests
@Test(arguments: ["election", "sports", "crypto"])
func search_returnsResults(query: String) async throws {
    let results = try await sut.search(query: query)
    #expect(!results.isEmpty)
}

// Tags for Organization
extension Tag {
    @Tag static var critical: Self
    @Tag static var network: Self
}

@Test(.tags(.critical, .network))
func apiCall_succeeds() async throws { }

// Traits for Configuration
@Test(.timeLimit(.minutes(1)))
func longRunningOperation_completesInTime() async { }

@Test(.disabled("Bug #123 - Fix pending"))
func knownBrokenFeature() { }

// Confirmation (replaces XCTest expectation/fulfill)
@Test func publisher_emitsValues() async {
    await confirmation("Received value") { confirm in
        let cancellable = viewModel.$items
            .dropFirst()
            .sink { _ in confirm() }

        viewModel.refresh()
        // confirmation must be called within timeout
    }
}

// Multiple confirmations
@Test func multipleEvents() async {
    await confirmation("Events received", expectedCount: 3) { confirm in
        // confirm() must be called exactly 3 times to pass
    }
}
```

#### 5. Test Types → iOS Mapping

```markdown
### Unit Tests (Swift Testing) - Mandatory
- Test ViewModels, Reducers, UseCases, Repositories
- Use protocol-based mocking
- Test in isolation with dependency injection
- Leverage parameterized tests for multiple inputs

### Integration Tests (Swift Testing) - Mandatory
- Test API client with real/mock server
- Test CoreData/SwiftData operations
- Test Combine/async streams

### UI Tests (XCUITest) - For Critical Flows
- Test complete user journeys
- Use accessibility identifiers
- Test on multiple device sizes

**⚠️ Important: XCUITest Target Separation**
- XCUITest still uses **XCTest framework** (not Swift Testing)
- **Cannot mix** Swift Testing and XCUITest in same target
- Recommended structure:
  - `MyAppTests` target: Swift Testing (Unit + Integration)
  - `MyAppUITests` target: XCTest (XCUITest only)
```

#### 6. Mocking → Swift Protocol-based

```swift
// Protocol
protocol MarketRepositoryProtocol: Sendable {
    func searchByVector(_ embedding: [Float]) async throws -> [Market]
}

// Mock for Swift Testing
final class MockMarketRepository: MarketRepositoryProtocol, @unchecked Sendable {
    var searchResult: Result<[Market], Error> = .success([])
    var searchCallCount = 0

    func searchByVector(_ embedding: [Float]) async throws -> [Market] {
        searchCallCount += 1
        return try searchResult.get()
    }
}

// Usage in test
@Suite struct MarketSearchTests {
    @Test func search_callsRepository() async throws {
        let mockRepo = MockMarketRepository()
        mockRepo.searchResult = .success([Market.stub()])
        let sut = MarketSearchUseCase(repository: mockRepo)

        _ = try await sut.search(query: "test")

        #expect(mockRepo.searchCallCount == 1)
    }
}

// Alternative: Actor-based Mock (Thread-safe, no @unchecked needed)
actor ActorMockMarketRepository: MarketRepositoryProtocol {
    var searchResult: Result<[Market], Error> = .success([])
    private(set) var searchCallCount = 0

    func searchByVector(_ embedding: [Float]) async throws -> [Market] {
        searchCallCount += 1
        return try searchResult.get()
    }

    func configure(result: Result<[Market], Error>) {
        searchResult = result
    }
}

// Usage with Actor mock
@Test func search_withActorMock() async throws {
    let mockRepo = ActorMockMarketRepository()
    await mockRepo.configure(result: .success([Market.stub()]))
    let sut = MarketSearchUseCase(repository: mockRepo)

    _ = try await sut.search(query: "test")

    let callCount = await mockRepo.searchCallCount
    #expect(callCount == 1)
}
```

#### 7. Coverage Report → Xcode

```bash
# Run tests with coverage
xcodebuild test \
  -scheme MyApp \
  -destination 'platform=iOS Simulator,name=iPhone 15' \
  -enableCodeCoverage YES

# Generate report (using xcov)
xcov --scheme MyApp --output_directory coverage

# Or using xccov (built-in)
xcrun xccov view --report --json Build/Logs/Test/*.xcresult
```

#### 8. iOS-Specific Edge Cases (Add)

```markdown
### Memory & Lifecycle
- View appears/disappears during async operation
- App backgrounded during network request
- Low memory warning handling

### Concurrency
- Task cancellation mid-operation
- Multiple concurrent requests
- Main actor isolation
- Sendable compliance

### Device Specific
- Different screen sizes (iPhone SE → iPad Pro)
- Dark mode / Light mode
- Dynamic Type sizes
- Orientation changes
```

#### 9. Test Smells → Swift Testing Version

```swift
// ❌ Testing Implementation Details
#expect(viewModel.internalState == .loading)

// ✅ Test Observable Behavior
#expect(viewModel.isLoading)
#expect(viewModel.displayedItems.count == 5)

// ❌ Dependent Tests (shared state between tests)
// ✅ Independent Tests - each @Test gets fresh instance via init()

// ❌ No assertions
@Test func something() {
    _ = sut.doSomething()  // No #expect!
}

// ✅ Explicit expectations
@Test func something_returnsExpectedValue() {
    let result = sut.doSomething()
    #expect(result == expectedValue)
}

// ❌ Flaky Test (non-deterministic)
@Test func network_succeeds() async throws {
    let result = try await realNetworkCall()  // Depends on actual network
    #expect(result != nil)
}

// ✅ Deterministic Test
@Test func network_succeeds() async throws {
    let mockNetwork = MockNetworkClient()
    mockNetwork.response = .success(MockData.user)
    let sut = UserService(network: mockNetwork)
    let result = try await sut.fetchUser()
    #expect(result != nil)
}

// ❌ Too many assertions (hard to identify failure)
@Test func user_isValid() {
    #expect(user.name == "John")
    #expect(user.age == 30)
    #expect(user.email.contains("@"))
    #expect(user.isActive)
    #expect(user.roles.contains(.admin))
}

// ✅ Focused tests with descriptive names
@Test func user_hasValidName() { #expect(user.name == "John") }
@Test func user_hasValidAge() { #expect(user.age == 30) }
@Test func user_hasValidEmail() { #expect(user.email.contains("@")) }
```

---

## 4. code-reviewer.md Analysis

(Next learning scheduled)

---
