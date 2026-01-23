# iOS Testing Requirements

> Essential testing requirements and guidelines for iOS applications
> Supports: XCTest, Swift Testing (Xcode 16+), XCUITest

---

## Minimum Test Coverage: 80%

| Layer | Target | Notes |
|-------|--------|-------|
| Domain (UseCase, Entity) | 90%+ | Core business logic |
| Data (Repository, DTO) | 80%+ | Data transformation/mapping |
| Presentation (ViewModel) | 70%+ | UI logic |
| View (SwiftUI/UIKit) | 0-30% | Use snapshot tests instead |

### Coverage Exclusions
- Generated code (Core Data entities, SwiftGen)
- Simple DTOs with no logic
- SwiftUI View body
- `#Preview` code

---

## Test Types (ALL required)

### 1. Unit Tests - Business logic, utilities

```swift
// Swift Testing (Xcode 16+, recommended)
@Suite("UserService Tests")
struct UserServiceTests {
    @Test("Fetch user returns valid data")
    func fetchUser() async throws {
        let sut = UserService(repository: MockRepository())
        let user = try await sut.fetchUser(id: "123")
        #expect(user.name == "Test User")
    }

    @Test("Invalid ID throws error", arguments: ["", " ", "invalid"])
    func invalidId(id: String) async {
        let sut = UserService(repository: MockRepository())
        await #expect(throws: UserError.invalidId) {
            try await sut.fetchUser(id: id)
        }
    }
}

// XCTest (Traditional)
class UserServiceTests: XCTestCase {
    func test_fetchUser_withValidId_returnsUser() async throws {
        let sut = UserService(repository: MockRepository())
        let user = try await sut.fetchUser(id: "123")
        XCTAssertEqual(user.name, "Test User")
    }
}
```

### 2. Integration Tests - API, Database

```swift
@Suite("Persistence Integration")
struct PersistenceTests {
    @Test("Save and fetch user")
    func saveAndFetch() async throws {
        let container = try ModelContainer(
            for: User.self,
            configurations: .init(isStoredInMemoryOnly: true)
        )
        let context = container.mainContext

        let user = User(name: "Test")
        context.insert(user)
        try context.save()

        let fetched = try context.fetch(FetchDescriptor<User>())
        #expect(fetched.count == 1)
    }
}
```

### 3. E2E Tests - XCUITest

```swift
class LoginE2ETests: XCTestCase {
    var app: XCUIApplication!

    override func setUpWithError() throws {
        continueAfterFailure = false
        app = XCUIApplication()
        app.launchArguments = ["--uitesting"]
        app.launch()
    }

    func test_loginFlow_withValidCredentials_showsHome() {
        // Given
        let emailField = app.textFields["email"]
        let passwordField = app.secureTextFields["password"]
        let loginButton = app.buttons["login"]

        // When
        emailField.tap()
        emailField.typeText("test@example.com")
        passwordField.tap()
        passwordField.typeText("password123")
        loginButton.tap()

        // Then
        XCTAssertTrue(app.staticTexts["Welcome"].waitForExistence(timeout: 5))
    }
}
```

---

## Test-Driven Development (TDD)

MANDATORY workflow:

1. **Write test first (RED)**
   - Swift Testing: write `@Test func`
   - XCTest: write `test_` prefixed function
   - Mock dependencies via Protocol

2. **Run test - MUST FAIL**
   - `Cmd+U` or `xcodebuild test`
   - Compile errors count as failure

3. **Write MINIMAL implementation (GREEN)**
   - Only write enough code to pass the test

4. **Run test - MUST PASS**

5. **Refactor (IMPROVE)**
   - Remove duplication, improve naming
   - Tests must continue to pass

6. **Verify coverage (80%+)**
   - Xcode: Product > Test > Code Coverage
   - CI: `xcodebuild -enableCodeCoverage YES`

---

## Test Naming Convention

```swift
// XCTest: SUT_Action_ExpectedResult
func test_fetchUser_withInvalidId_throwsError() { }
func test_login_whenNetworkFails_showsAlert() { }

// Swift Testing: Natural language description
@Test("Login with valid credentials returns user")
func loginSuccess() { }

@Test("Fetch user throws error when ID is empty")
func fetchUserInvalidId() { }
```

---

## Mock/Stub Strategy (Protocol-based DI)

```swift
// 1. Define Protocol
protocol UserRepositoryProtocol {
    func fetchUser(id: String) async throws -> User
}

// 2. Stub (returns fixed values)
struct StubUserRepository: UserRepositoryProtocol {
    var result: Result<User, Error> = .success(User(name: "Stub"))

    func fetchUser(id: String) async throws -> User {
        try result.get()
    }
}

// 3. Mock (verifies calls)
class MockAnalytics: AnalyticsProtocol {
    private(set) var trackedEvents: [String] = []

    func track(event: String) {
        trackedEvents.append(event)
    }
}

// 4. Fake (simplified real implementation)
class FakeUserRepository: UserRepositoryProtocol {
    private var storage: [String: User] = [:]

    func save(_ user: User) { storage[user.id] = user }
    func fetchUser(id: String) async throws -> User {
        guard let user = storage[id] else { throw NotFoundError() }
        return user
    }
}

// 5. Injection via DI
struct UserService {
    private let repository: UserRepositoryProtocol

    init(repository: UserRepositoryProtocol = UserRepository()) {
        self.repository = repository
    }
}
```

---

## Async Testing Patterns

```swift
// Swift Testing (recommended)
@Test func asyncOperation() async throws {
    let result = try await sut.performAsync()
    #expect(result.isSuccess)
}

// XCTest with async/await
func test_asyncOperation() async throws {
    let result = try await sut.performAsync()
    XCTAssertTrue(result.isSuccess)
}

// @MainActor testing
@Suite("ViewModel Tests")
@MainActor
struct ViewModelTests {
    @Test func loadData_setsLoadingTrue() async {
        let sut = ViewModel()
        Task { await sut.loadData() }
        try? await Task.sleep(for: .milliseconds(10))
        #expect(sut.isLoading == true)
    }
}
```

---

## Performance Testing

```swift
// Only supported in XCTest (not Swift Testing)
func test_imageProcessing_performance() {
    let image = UIImage(named: "large_image")!
    let processor = ImageProcessor()

    measure(metrics: [
        XCTClockMetric(),     // Execution time
        XCTCPUMetric(),       // CPU usage
        XCTMemoryMetric()     // Memory usage
    ]) {
        _ = processor.resize(image, to: CGSize(width: 100, height: 100))
    }
}
```

---

## Snapshot Testing

```swift
// Using swift-snapshot-testing library
import SnapshotTesting

final class LoginViewSnapshotTests: XCTestCase {
    func test_loginView_defaultState() {
        let view = LoginView(viewModel: .preview)

        assertSnapshot(
            of: view,
            as: .image(layout: .device(config: .iPhone13Pro))
        )
    }

    func test_loginView_darkMode() {
        let view = LoginView(viewModel: .preview)
            .environment(\.colorScheme, .dark)

        assertSnapshot(
            of: view,
            as: .image(layout: .device(config: .iPhone13Pro))
        )
    }
}
```

---

## Test Data Management (Fixtures)

```swift
enum TestFixtures {
    static let validUser = User(
        id: "test-123",
        name: "Test User",
        email: "test@example.com"
    )

    static func user(
        id: String = "test-123",
        name: String = "Test User"
    ) -> User {
        User(id: id, name: name, email: "\(name.lowercased())@test.com")
    }
}

// Managing JSON fixtures
extension Bundle {
    func loadJSON<T: Decodable>(_ filename: String) throws -> T {
        let url = self.url(forResource: filename, withExtension: "json")!
        let data = try Data(contentsOf: url)
        return try JSONDecoder().decode(T.self, from: data)
    }
}
```

---

## CI/CD Commands

```bash
# Unit + Integration Tests
xcodebuild test \
    -scheme "MyApp" \
    -destination "platform=iOS Simulator,name=iPhone 16" \
    -enableCodeCoverage YES \
    -resultBundlePath ./TestResults.xcresult

# UI Tests (recommended to run separately)
xcodebuild test \
    -scheme "MyAppUITests" \
    -destination "platform=iOS Simulator,name=iPhone 16" \
    -testPlan "UITests"

# Flaky test retry
xcodebuild test \
    -retry-tests-on-failure \
    -test-iterations 3
```

---

## Troubleshooting Test Failures

1. Use **swift-tdd-guide** agent
2. Check test isolation (no shared state between tests)
3. Verify mocks are correct (protocol conformance)
4. Fix implementation, not tests (unless tests are wrong)
5. Check async timing issues (waitForExistence, Task.sleep)

---

## Agent Support

- **swift-tdd-guide** - TDD workflow guidance, test writing support
- **xcuitest-runner** - XCUITest E2E testing specialist

---

## Testing Checklist

### Test Structure
- [ ] Test naming: `SUT_Action_ExpectedResult` or natural language (@Test)
- [ ] One assertion per test (when possible)
- [ ] Arrange-Act-Assert pattern followed
- [ ] Test isolation (no shared state)

### Test Doubles
- [ ] Protocol-based dependency injection
- [ ] Stub/Mock/Fake used appropriately
- [ ] Mock only for call verification

### Async Testing
- [ ] Prefer async/await (XCTestExpectation is legacy)
- [ ] @MainActor tests also marked @MainActor
- [ ] Task cancellation tested

### Coverage
- [ ] Domain layer 90%+
- [ ] Data layer 80%+
- [ ] Generated code excluded

### Performance & Snapshot
- [ ] Performance tests for critical paths (XCTMetric)
- [ ] Snapshot tests for UI regression prevention
- [ ] Both dark/light mode tested

### CI/CD
- [ ] Tests grouped by Test Plan
- [ ] Flaky test retry configured
- [ ] Parallel execution optimized
