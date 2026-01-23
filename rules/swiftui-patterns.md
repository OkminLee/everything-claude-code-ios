# SwiftUI Common Patterns

## API Response Pattern

```swift
struct APIResponse<T: Codable>: Codable {
    let success: Bool
    let data: T?
    let error: String?
    let meta: PageMeta?
}

struct PageMeta: Codable {
    let total: Int
    let page: Int
    let limit: Int
}

// Usage with async/await
func fetch<T: Codable>(_ endpoint: String) async throws -> T {
    let (data, response) = try await URLSession.shared.data(from: url)
    guard let httpResponse = response as? HTTPURLResponse,
          (200...299).contains(httpResponse.statusCode) else {
        throw APIError.invalidResponse
    }
    return try JSONDecoder().decode(APIResponse<T>.self, from: data).data!
}
```

## Debounce Pattern

### Using Combine (iOS 13+)

```swift
import Combine

final class SearchViewModel: ObservableObject {
    @Published var searchText = ""
    @Published private(set) var results: [Item] = []

    private var cancellables = Set<AnyCancellable>()

    init() {
        $searchText
            .debounce(for: .milliseconds(300), scheduler: RunLoop.main)
            .removeDuplicates()
            .sink { [weak self] text in
                guard let self else { return }
                Task { await self.performSearch(text) }
            }
            .store(in: &cancellables)
    }

    deinit {
        cancellables.removeAll()
    }
}
```

### Using Task (iOS 15+)

```swift
@Observable
final class SearchViewModel {
    var searchText = "" {
        didSet { debounceSearch() }
    }
    private(set) var results: [Item] = []

    private var searchTask: Task<Void, Never>?

    private func debounceSearch() {
        searchTask?.cancel()
        searchTask = Task {
            try? await Task.sleep(for: .milliseconds(300))
            guard !Task.isCancelled else { return }
            await performSearch(searchText)
        }
    }
}
```

## Repository Pattern

```swift
protocol Repository<Entity> {
    associatedtype Entity
    associatedtype ID: Hashable

    func findAll(filter: QueryFilter?) async throws -> [Entity]
    func findById(_ id: ID) async throws -> Entity?
    func create(_ entity: Entity) async throws -> Entity
    func update(_ entity: Entity) async throws -> Entity
    func delete(_ id: ID) async throws
}

// Implementation example
final class UserRepository: Repository {
    typealias Entity = User
    typealias ID = UUID

    private let apiClient: APIClient
    private let cache: UserCache

    func findById(_ id: UUID) async throws -> User? {
        if let cached = cache.get(id) { return cached }
        let user = try await apiClient.fetchUser(id: id)
        cache.set(user)
        return user
    }
    // ... other methods
}
```

## ViewModifier Pattern

### Custom ViewModifier

```swift
struct CardStyle: ViewModifier {
    let cornerRadius: CGFloat
    let shadowRadius: CGFloat

    init(cornerRadius: CGFloat = 12, shadowRadius: CGFloat = 4) {
        self.cornerRadius = cornerRadius
        self.shadowRadius = shadowRadius
    }

    func body(content: Content) -> some View {
        content
            .background(.background)
            .clipShape(RoundedRectangle(cornerRadius: cornerRadius))
            .shadow(color: .black.opacity(0.1), radius: shadowRadius, y: 2)
    }
}

extension View {
    func cardStyle(cornerRadius: CGFloat = 12, shadowRadius: CGFloat = 4) -> some View {
        modifier(CardStyle(cornerRadius: cornerRadius, shadowRadius: shadowRadius))
    }
}
```

### Conditional Modifier

```swift
extension View {
    @ViewBuilder
    func `if`<Content: View>(_ condition: Bool, transform: (Self) -> Content) -> some View {
        if condition {
            transform(self)
        } else {
            self
        }
    }

    @ViewBuilder
    func ifLet<Value, Content: View>(_ value: Value?, transform: (Self, Value) -> Content) -> some View {
        if let value {
            transform(self, value)
        } else {
            self
        }
    }
}

// Usage
Text("Hello")
    .if(isHighlighted) { $0.foregroundStyle(.red) }
    .ifLet(backgroundColor) { view, color in view.background(color) }
```

## PreferenceKey Pattern

### Size Reader

```swift
struct SizePreferenceKey: PreferenceKey {
    static var defaultValue: CGSize = .zero
    static func reduce(value: inout CGSize, nextValue: () -> CGSize) {
        value = nextValue()
    }
}

extension View {
    func readSize(_ onChange: @escaping (CGSize) -> Void) -> some View {
        background(
            GeometryReader { geometry in
                Color.clear
                    .preference(key: SizePreferenceKey.self, value: geometry.size)
            }
        )
        .onPreferenceChange(SizePreferenceKey.self, perform: onChange)
    }
}
```

### Scroll Offset Reader

```swift
struct ScrollOffsetPreferenceKey: PreferenceKey {
    static var defaultValue: CGFloat = 0
    static func reduce(value: inout CGFloat, nextValue: () -> CGFloat) {
        value = nextValue()
    }
}

struct ScrollOffsetModifier: ViewModifier {
    let coordinateSpace: String
    @Binding var offset: CGFloat

    func body(content: Content) -> some View {
        content
            .background(
                GeometryReader { proxy in
                    Color.clear
                        .preference(
                            key: ScrollOffsetPreferenceKey.self,
                            value: proxy.frame(in: .named(coordinateSpace)).minY
                        )
                }
            )
            .onPreferenceChange(ScrollOffsetPreferenceKey.self) { offset = $0 }
    }
}
```

## Navigation Pattern

### NavigationStack (iOS 16+)

```swift
enum Route: Hashable {
    case detail(Item)
    case settings
    case profile(userId: String)
}

struct ContentView: View {
    @State private var path = NavigationPath()

    var body: some View {
        NavigationStack(path: $path) {
            ListView()
                .navigationDestination(for: Route.self) { route in
                    switch route {
                    case .detail(let item):
                        DetailView(item: item)
                    case .settings:
                        SettingsView()
                    case .profile(let userId):
                        ProfileView(userId: userId)
                    }
                }
        }
    }

    func navigate(to route: Route) {
        path.append(route)
    }

    func popToRoot() {
        path.removeLast(path.count)
    }
}
```

### Coordinator Pattern

```swift
@MainActor
protocol Coordinator: AnyObject {
    var navigationController: UINavigationController { get }
    func start()
}

@MainActor
final class AppCoordinator: Coordinator {
    let navigationController: UINavigationController
    private var childCoordinators: [Coordinator] = []

    init(navigationController: UINavigationController) {
        self.navigationController = navigationController
    }

    func start() {
        showHome()
    }

    private func showHome() {
        let viewModel = HomeViewModel()
        viewModel.onItemSelected = { [weak self] item in
            self?.showDetail(item)
        }
        let vc = HomeViewController(viewModel: viewModel)
        navigationController.pushViewController(vc, animated: true)
    }
}
```

## State Management

### @Observable (iOS 17+)

```swift
import Observation

@Observable
final class AppState {
    var user: User?
    var isAuthenticated: Bool { user != nil }
    var settings = Settings()

    func logout() {
        user = nil
    }
}

// Usage in View
struct ContentView: View {
    @State private var appState = AppState()

    var body: some View {
        MainView()
            .environment(appState)
    }
}

struct ProfileView: View {
    @Environment(AppState.self) private var appState

    var body: some View {
        if let user = appState.user {
            Text(user.name)
        }
    }
}
```

### @Bindable (iOS 17+)

```swift
@Observable
final class ProfileViewModel {
    var name: String = ""
    var email: String = ""
    var isNotificationsEnabled: Bool = true
}

struct ProfileView: View {
    @Bindable var viewModel: ProfileViewModel

    var body: some View {
        Form {
            TextField("Name", text: $viewModel.name)
            TextField("Email", text: $viewModel.email)
            Toggle("Notifications", isOn: $viewModel.isNotificationsEnabled)
        }
    }
}
```

### TCA Pattern (The Composable Architecture)

```swift
import ComposableArchitecture

@Reducer
struct CounterFeature {
    @ObservableState
    struct State: Equatable {
        var count = 0
    }

    enum Action {
        case incrementTapped
        case decrementTapped
    }

    var body: some ReducerOf<Self> {
        Reduce { state, action in
            switch action {
            case .incrementTapped:
                state.count += 1
                return .none
            case .decrementTapped:
                state.count -= 1
                return .none
            }
        }
    }
}

struct CounterView: View {
    let store: StoreOf<CounterFeature>

    var body: some View {
        HStack {
            Button("-") { store.send(.decrementTapped) }
            Text("\(store.count)")
            Button("+") { store.send(.incrementTapped) }
        }
    }
}
```

## Actor & Concurrency Pattern

### Actor Pattern

```swift
actor ImageCache {
    static let shared = ImageCache()

    private var cache: [URL: UIImage] = [:]
    private var inFlightTasks: [URL: Task<UIImage?, Never>] = [:]

    func image(for url: URL) async -> UIImage? {
        if let cached = cache[url] {
            return cached
        }

        if let existingTask = inFlightTasks[url] {
            return await existingTask.value
        }

        let task = Task<UIImage?, Never> {
            guard let (data, _) = try? await URLSession.shared.data(from: url),
                  let image = UIImage(data: data) else {
                return nil
            }
            return image
        }

        inFlightTasks[url] = task
        let image = await task.value
        inFlightTasks[url] = nil

        if let image {
            cache[url] = image
        }

        return image
    }

    func clearCache() {
        cache.removeAll()
    }
}
```

### Sendable Conformance

```swift
struct UserDTO: Sendable, Codable {
    let id: UUID
    let name: String
    let email: String
}

final class UserService: Sendable {
    private let urlSession: URLSession

    init(urlSession: URLSession = .shared) {
        self.urlSession = urlSession
    }

    func fetchUser(id: UUID) async throws -> UserDTO {
        let url = URL(string: "https://api.example.com/users/\(id)")!
        let (data, _) = try await urlSession.data(from: url)
        return try JSONDecoder().decode(UserDTO.self, from: data)
    }
}
```

### MainActor Isolation

```swift
@MainActor
@Observable
final class UserListViewModel {
    private(set) var users: [User] = []
    private(set) var isLoading = false
    private(set) var error: Error?

    private let userService: UserService

    init(userService: UserService = UserService()) {
        self.userService = userService
    }

    func loadUsers() async {
        isLoading = true
        error = nil

        do {
            users = try await userService.fetchAllUsers()
        } catch {
            self.error = error
        }

        isLoading = false
    }
}
```

## Memory Management

### Weak Self in Closures

```swift
final class TimerViewModel: ObservableObject {
    @Published var count = 0
    private var timer: Timer?

    func startTimer() {
        // Correct: weak self to avoid retain cycle
        timer = Timer.scheduledTimer(withTimeInterval: 1, repeats: true) { [weak self] _ in
            self?.count += 1
        }
    }

    func stopTimer() {
        timer?.invalidate()
        timer = nil
    }

    deinit {
        stopTimer()
    }
}
```

### Combine Memory Management

```swift
final class SearchViewModel: ObservableObject {
    @Published var searchText = ""
    @Published private(set) var results: [Item] = []

    private var cancellables = Set<AnyCancellable>()

    init() {
        $searchText
            .debounce(for: .milliseconds(300), scheduler: RunLoop.main)
            .removeDuplicates()
            .sink { [weak self] text in
                guard let self else { return }
                Task { await self.performSearch(text) }
            }
            .store(in: &cancellables)
    }

    deinit {
        cancellables.removeAll()
    }
}
```

### Task Cancellation

```swift
struct UserListView: View {
    @State private var viewModel = UserListViewModel()
    @State private var loadTask: Task<Void, Never>?

    var body: some View {
        List(viewModel.users) { user in
            UserRow(user: user)
        }
        .task {
            // .task auto-cancels when View disappears
            await viewModel.loadUsers()
        }
        .refreshable {
            await viewModel.loadUsers()
        }
        .onDisappear {
            loadTask?.cancel()
        }
    }
}
```

## Dependency Injection

### Environment-based DI

```swift
private struct APIClientKey: EnvironmentKey {
    static let defaultValue: APIClientProtocol = APIClient()
}

extension EnvironmentValues {
    var apiClient: APIClientProtocol {
        get { self[APIClientKey.self] }
        set { self[APIClientKey.self] = newValue }
    }
}

// Usage
struct ContentView: View {
    @Environment(\.apiClient) private var apiClient

    var body: some View {
        Button("Fetch") {
            Task { try await apiClient.fetch() }
        }
    }
}

// Testing with mock
#Preview {
    ContentView()
        .environment(\.apiClient, MockAPIClient())
}
```

### Factory Pattern

```swift
protocol DependencyContainer {
    func makeUserRepository() -> UserRepository
    func makeAuthService() -> AuthService
}

final class AppDependencyContainer: DependencyContainer {
    private lazy var apiClient = APIClient()
    private lazy var database = Database()

    func makeUserRepository() -> UserRepository {
        UserRepository(apiClient: apiClient, database: database)
    }

    func makeAuthService() -> AuthService {
        AuthService(apiClient: apiClient)
    }
}
```

## Error Handling Pattern

### Typed Errors

```swift
enum NetworkError: LocalizedError {
    case invalidURL
    case noConnection
    case timeout
    case serverError(statusCode: Int)
    case decodingFailed(Error)

    var errorDescription: String? {
        switch self {
        case .invalidURL: return "Invalid URL"
        case .noConnection: return "No internet connection"
        case .timeout: return "Request timed out"
        case .serverError(let code): return "Server error: \(code)"
        case .decodingFailed: return "Failed to parse response"
        }
    }

    var isRetryable: Bool {
        switch self {
        case .noConnection, .timeout: return true
        default: return false
        }
    }
}
```

## Loading State Pattern

```swift
enum LoadingState<T> {
    case idle
    case loading
    case loaded(T)
    case failed(Error)

    var isLoading: Bool {
        if case .loading = self { return true }
        return false
    }

    var value: T? {
        if case .loaded(let value) = self { return value }
        return nil
    }

    var error: Error? {
        if case .failed(let error) = self { return error }
        return nil
    }
}

// Generic Container View
struct AsyncContentView<Content: View, T>: View {
    let state: LoadingState<T>
    let retryAction: () async -> Void
    @ViewBuilder let content: (T) -> Content

    var body: some View {
        switch state {
        case .idle:
            Color.clear
                .task { await retryAction() }
        case .loading:
            ProgressView()
                .frame(maxWidth: .infinity, maxHeight: .infinity)
        case .loaded(let data):
            content(data)
        case .failed(let error):
            ContentUnavailableView {
                Label("Error", systemImage: "exclamationmark.triangle")
            } description: {
                Text(error.localizedDescription)
            } actions: {
                Button("Retry") {
                    Task { await retryAction() }
                }
            }
        }
    }
}
```

## Pagination Pattern

### Pagination State

```swift
struct PaginationState<T: Identifiable> {
    var items: [T] = []
    var currentPage = 1
    var hasMorePages = true
    var isLoadingMore = false

    mutating func appendPage(_ newItems: [T], hasMore: Bool) {
        items.append(contentsOf: newItems)
        currentPage += 1
        hasMorePages = hasMore
        isLoadingMore = false
    }

    mutating func reset() {
        items = []
        currentPage = 1
        hasMorePages = true
        isLoadingMore = false
    }
}
```

### Paginated ViewModel

```swift
@MainActor
@Observable
final class PaginatedListViewModel {
    private(set) var pagination = PaginationState<Item>()
    private(set) var initialLoadState: LoadingState<Void> = .idle

    private let repository: ItemRepository
    private let pageSize = 20

    func loadInitialPage() async {
        guard case .idle = initialLoadState else { return }

        initialLoadState = .loading
        pagination.reset()

        do {
            let result = try await repository.fetchItems(page: 1, limit: pageSize)
            pagination.appendPage(result.items, hasMore: result.hasMore)
            initialLoadState = .loaded(())
        } catch {
            initialLoadState = .failed(error)
        }
    }

    func loadMoreIfNeeded(currentItem: Item) async {
        guard let index = pagination.items.firstIndex(where: { $0.id == currentItem.id }),
              index >= pagination.items.count - 5,
              pagination.hasMorePages,
              !pagination.isLoadingMore else {
            return
        }

        pagination.isLoadingMore = true

        do {
            let result = try await repository.fetchItems(
                page: pagination.currentPage + 1,
                limit: pageSize
            )
            pagination.appendPage(result.items, hasMore: result.hasMore)
        } catch {
            pagination.isLoadingMore = false
        }
    }
}
```

### Paginated List View

```swift
struct PaginatedListView: View {
    @State private var viewModel: PaginatedListViewModel

    var body: some View {
        AsyncContentView(
            state: viewModel.initialLoadState,
            retryAction: { await viewModel.loadInitialPage() }
        ) { _ in
            List {
                ForEach(viewModel.pagination.items) { item in
                    ItemRow(item: item)
                        .task {
                            await viewModel.loadMoreIfNeeded(currentItem: item)
                        }
                }

                if viewModel.pagination.isLoadingMore {
                    HStack {
                        Spacer()
                        ProgressView()
                        Spacer()
                    }
                    .listRowSeparator(.hidden)
                }
            }
            .refreshable {
                viewModel.initialLoadState = .idle
                await viewModel.loadInitialPage()
            }
        }
    }
}
```

## Accessibility Pattern

### Accessible Components

```swift
struct AccessibleButton: View {
    let title: String
    let icon: String
    let action: () -> Void

    var body: some View {
        Button(action: action) {
            Label(title, systemImage: icon)
        }
        .accessibilityLabel(title)
        .accessibilityHint("Double tap to \(title.lowercased())")
        .accessibilityAddTraits(.isButton)
    }
}
```

### Dynamic Type Support

```swift
struct ScalableText: View {
    let text: String
    @ScaledMetric(relativeTo: .body) private var iconSize = 20

    var body: some View {
        HStack {
            Image(systemName: "info.circle")
                .frame(width: iconSize, height: iconSize)

            Text(text)
                .font(.body)
                .minimumScaleFactor(0.8)
        }
    }
}
```

### VoiceOver Support

```swift
struct ProductCard: View {
    let product: Product

    var body: some View {
        VStack(alignment: .leading) {
            AsyncImage(url: product.imageURL)
                .accessibilityHidden(true)

            Text(product.name)
                .font(.headline)

            Text(product.price, format: .currency(code: "KRW"))
                .font(.subheadline)
        }
        .accessibilityElement(children: .combine)
        .accessibilityLabel("\(product.name), \(product.price)원")
        .accessibilityHint(product.isInStock ? "In stock" : "Out of stock")
    }
}
```

### Reduce Motion Support

```swift
struct AnimatedView: View {
    @Environment(\.accessibilityReduceMotion) private var reduceMotion
    @State private var isAnimating = false

    var body: some View {
        Circle()
            .scaleEffect(isAnimating ? 1.2 : 1.0)
            .animation(
                reduceMotion ? .none : .easeInOut(duration: 0.5).repeatForever(),
                value: isAnimating
            )
            .onAppear { isAnimating = true }
    }
}
```

## ViewBuilder & Custom Containers

### Custom Container View

```swift
struct Card<Content: View>: View {
    let title: String
    @ViewBuilder let content: () -> Content

    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            Text(title)
                .font(.headline)

            content()
        }
        .padding()
        .cardStyle()
    }
}

// Usage
Card(title: "User Info") {
    Text("Name: John")
    Text("Email: john@example.com")
}
```

## Custom Layout (iOS 16+)

### Flow Layout (Tag Cloud)

```swift
struct FlowLayout: Layout {
    var spacing: CGFloat = 8

    func sizeThatFits(proposal: ProposedViewSize, subviews: Subviews, cache: inout ()) -> CGSize {
        let result = arrangeSubviews(proposal: proposal, subviews: subviews)
        return result.size
    }

    func placeSubviews(in bounds: CGRect, proposal: ProposedViewSize, subviews: Subviews, cache: inout ()) {
        let result = arrangeSubviews(proposal: proposal, subviews: subviews)

        for (index, placement) in result.placements.enumerated() {
            subviews[index].place(
                at: CGPoint(x: bounds.minX + placement.x, y: bounds.minY + placement.y),
                proposal: .unspecified
            )
        }
    }

    private func arrangeSubviews(proposal: ProposedViewSize, subviews: Subviews) -> (size: CGSize, placements: [CGPoint]) {
        let maxWidth = proposal.width ?? .infinity
        var placements: [CGPoint] = []
        var currentX: CGFloat = 0
        var currentY: CGFloat = 0
        var lineHeight: CGFloat = 0
        var totalHeight: CGFloat = 0

        for subview in subviews {
            let size = subview.sizeThatFits(.unspecified)

            if currentX + size.width > maxWidth, currentX > 0 {
                currentX = 0
                currentY += lineHeight + spacing
                lineHeight = 0
            }

            placements.append(CGPoint(x: currentX, y: currentY))
            lineHeight = max(lineHeight, size.height)
            currentX += size.width + spacing
            totalHeight = currentY + lineHeight
        }

        return (CGSize(width: maxWidth, height: totalHeight), placements)
    }
}

// Usage
struct TagCloudView: View {
    let tags: [String]

    var body: some View {
        FlowLayout(spacing: 8) {
            ForEach(tags, id: \.self) { tag in
                Text(tag)
                    .padding(.horizontal, 12)
                    .padding(.vertical, 6)
                    .background(.blue.opacity(0.1))
                    .clipShape(Capsule())
            }
        }
    }
}
```

## iOS 17+ Modern APIs

### onChange with oldValue

```swift
struct SearchView: View {
    @State private var searchText = ""

    var body: some View {
        TextField("Search", text: $searchText)
            .onChange(of: searchText) { oldValue, newValue in
                if oldValue.isEmpty && !newValue.isEmpty {
                    analytics.trackSearchStarted()
                }
            }
    }
}
```

### contentMargins

```swift
ScrollView {
    LazyVStack {
        // content
    }
}
.contentMargins(.horizontal, 16, for: .scrollContent)
.contentMargins(.vertical, 8, for: .scrollIndicators)
```

### scrollTargetBehavior

```swift
ScrollView(.horizontal) {
    LazyHStack(spacing: 16) {
        ForEach(cards) { card in
            CardView(card: card)
                .containerRelativeFrame(.horizontal, count: 1, spacing: 16)
        }
    }
    .scrollTargetLayout()
}
.scrollTargetBehavior(.viewAligned)
.scrollIndicators(.hidden)
```

## Preview Pattern

```swift
#Preview("Default State") {
    ItemListView()
        .environment(MockAppState.default)
}

#Preview("Loading State") {
    ItemListView()
        .environment(MockAppState.loading)
}

#Preview("Error State") {
    ItemListView()
        .environment(MockAppState.error)
}

#Preview("Dark Mode", traits: .sizeThatFitsLayout) {
    ItemRow(item: .mock)
        .preferredColorScheme(.dark)
}

#Preview("Large Text", traits: .sizeThatFitsLayout) {
    ItemRow(item: .mock)
        .environment(\.sizeCategory, .accessibilityExtraExtraLarge)
}

// Mock Helper
enum MockAppState {
    static var `default`: AppState {
        let state = AppState()
        state.user = .mock
        return state
    }

    static var loading: AppState {
        let state = AppState()
        state.isLoading = true
        return state
    }
}

extension User {
    static var mock: User {
        User(id: UUID(), name: "Test User", email: "test@example.com")
    }
}
```

## Task Modifier Best Practices

```swift
struct DataView: View {
    @State private var viewModel = DataViewModel()

    var body: some View {
        List(viewModel.items) { item in
            ItemRow(item: item)
        }
        // Cancels previous task when id changes
        .task(id: viewModel.filterID) {
            await viewModel.loadItems()
        }
        // Runs only once on first appear
        .task {
            await viewModel.loadInitialData()
        }
    }
}

// Task Priority
struct ImportantDataView: View {
    @State private var viewModel = DataViewModel()

    var body: some View {
        ContentView()
            .task(priority: .userInitiated) {
                await viewModel.loadUserRequestedData()
            }
            .task(priority: .background) {
                await viewModel.syncInBackground()
            }
    }
}
```

## Skeleton Projects

When implementing new iOS functionality:

1. **Search for proven templates**
   - Apple's sample code (developer.apple.com/sample-code)
   - Swift Package Index (swiftpackageindex.com)
   - GitHub iOS templates with recent activity

2. **Evaluate with parallel agents**
   - Security assessment (dependencies audit)
   - iOS version compatibility check
   - Architecture fit analysis
   - Implementation complexity scoring

3. **Use as foundation**
   - Xcode project templates
   - SPM package templates
   - Feature module templates

4. **Iterate within structure**
   - Maintain consistent patterns
   - Follow established conventions
   - Document deviations

### Recommended Template Sources

| Source | Best For |
|--------|----------|
| Apple Sample Code | New APIs, platform features |
| swift-composable-architecture | TCA-based apps |
| pointfreeco/isowords | Complex SwiftUI apps |
| tca-case-paths | Navigation patterns |
