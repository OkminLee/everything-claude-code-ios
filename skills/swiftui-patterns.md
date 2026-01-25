---
name: swiftui-patterns
description: SwiftUI development patterns for view composition, state management, performance optimization, and UI best practices.
---

# SwiftUI Development Patterns

Modern SwiftUI patterns for building performant, maintainable user interfaces.

## View Composition Patterns

### Basic Composition

```swift
// ✅ GOOD: Component composition
struct Card<Content: View>: View {
    let content: Content

    init(@ViewBuilder content: () -> Content) {
        self.content = content()
    }

    var body: some View {
        VStack(alignment: .leading, spacing: 0) {
            content
        }
        .background(Color(.systemBackground))
        .cornerRadius(12)
        .shadow(radius: 2)
    }
}

struct CardHeader: View {
    let title: String

    var body: some View {
        Text(title)
            .font(.headline)
            .padding()
    }
}

struct CardBody<Content: View>: View {
    let content: Content

    init(@ViewBuilder content: () -> Content) {
        self.content = content()
    }

    var body: some View {
        content
            .padding([.horizontal, .bottom])
    }
}

// Usage
Card {
    CardHeader(title: "Market Overview")
    CardBody {
        Text("Current market status and metrics")
    }
}
```

### Compound Components with Environment

```swift
// Context sharing via Environment
struct TabsConfiguration {
    var activeTab: String
    var setActiveTab: (String) -> Void
}

private struct TabsConfigurationKey: EnvironmentKey {
    static let defaultValue: TabsConfiguration? = nil
}

extension EnvironmentValues {
    var tabsConfiguration: TabsConfiguration? {
        get { self[TabsConfigurationKey.self] }
        set { self[TabsConfigurationKey.self] = newValue }
    }
}

struct Tabs<Content: View>: View {
    @State private var activeTab: String
    let content: Content

    init(defaultTab: String, @ViewBuilder content: () -> Content) {
        self._activeTab = State(initialValue: defaultTab)
        self.content = content()
    }

    var body: some View {
        content
            .environment(\.tabsConfiguration, TabsConfiguration(
                activeTab: activeTab,
                setActiveTab: { activeTab = $0 }
            ))
    }
}

struct TabButton: View {
    let id: String
    let title: String
    @Environment(\.tabsConfiguration) private var config

    var body: some View {
        Button(title) {
            config?.setActiveTab(id)
        }
        .foregroundColor(config?.activeTab == id ? .accentColor : .secondary)
    }
}

// Usage
Tabs(defaultTab: "overview") {
    HStack {
        TabButton(id: "overview", title: "Overview")
        TabButton(id: "details", title: "Details")
    }
}
```

### ViewBuilder Closures (Render Props Pattern)

```swift
// Generic data loading view
struct DataLoader<T, Content: View, Loading: View, Failure: View>: View {
    let load: () async throws -> T
    @ViewBuilder let content: (T) -> Content
    @ViewBuilder let loading: () -> Loading
    @ViewBuilder let failure: (Error) -> Failure

    @State private var data: T?
    @State private var isLoading = true
    @State private var error: Error?

    var body: some View {
        Group {
            if isLoading {
                loading()
            } else if let error {
                failure(error)
            } else if let data {
                content(data)
            }
        }
        .task {
            await loadData()
        }
    }

    private func loadData() async {
        isLoading = true
        error = nil

        do {
            data = try await load()
        } catch {
            self.error = error
        }

        isLoading = false
    }
}

// Usage
DataLoader(
    load: { try await api.fetchMarkets() },
    content: { markets in
        MarketListView(markets: markets)
    },
    loading: {
        ProgressView()
    },
    failure: { error in
        ErrorView(error: error)
    }
)
```

## Property Wrappers (Custom Hooks Pattern)

### Toggle State

```swift
// ✅ Reusable toggle state
@propertyWrapper
struct Toggle: DynamicProperty {
    @State private var value: Bool

    init(wrappedValue: Bool = false) {
        self._value = State(initialValue: wrappedValue)
    }

    var wrappedValue: Bool {
        get { value }
        nonmutating set { value = newValue }
    }

    var projectedValue: Binding<Bool> {
        Binding(
            get: { value },
            set: { value = $0 }
        )
    }

    func toggle() {
        value.toggle()
    }
}

// Usage
struct ContentView: View {
    @Toggle var isOpen

    var body: some View {
        Button("Toggle: \(isOpen ? "Open" : "Closed")") {
            _isOpen.toggle()
        }
    }
}
```

### Debounced Value

```swift
// ✅ Debounce input changes
@propertyWrapper
struct Debounced<Value>: DynamicProperty {
    @State private var value: Value
    @State private var debouncedValue: Value
    private let delay: Duration

    init(wrappedValue: Value, delay: Duration = .milliseconds(500)) {
        self._value = State(initialValue: wrappedValue)
        self._debouncedValue = State(initialValue: wrappedValue)
        self.delay = delay
    }

    var wrappedValue: Value {
        get { value }
        nonmutating set { value = newValue }
    }

    var projectedValue: Value {
        debouncedValue
    }

    func update() {
        Task {
            try? await Task.sleep(for: delay)
            await MainActor.run {
                debouncedValue = value
            }
        }
    }
}

// Usage
struct SearchView: View {
    @Debounced var searchQuery = ""

    var body: some View {
        TextField("Search", text: $searchQuery)
            .onChange(of: searchQuery) {
                _searchQuery.update()
            }
            .onChange(of: $searchQuery) { query in
                performSearch(query: query)
            }
    }
}
```

## State Management Patterns

### Observable with Reducer

```swift
// State definition
struct MarketState {
    var markets: [Market] = []
    var selectedMarket: Market?
    var isLoading = false
    var error: Error?
}

// Actions
enum MarketAction {
    case setMarkets([Market])
    case selectMarket(Market)
    case setLoading(Bool)
    case setError(Error?)
}

// Reducer
func marketReducer(state: inout MarketState, action: MarketAction) {
    switch action {
    case .setMarkets(let markets):
        state.markets = markets
    case .selectMarket(let market):
        state.selectedMarket = market
    case .setLoading(let loading):
        state.isLoading = loading
    case .setError(let error):
        state.error = error
    }
}

// Observable store
@Observable
final class MarketStore {
    private(set) var state = MarketState()

    func dispatch(_ action: MarketAction) {
        marketReducer(state: &state, action: action)
    }

    func loadMarkets() async {
        dispatch(.setLoading(true))
        dispatch(.setError(nil))

        do {
            let markets = try await api.fetchMarkets()
            dispatch(.setMarkets(markets))
        } catch {
            dispatch(.setError(error))
        }

        dispatch(.setLoading(false))
    }
}

// Usage
struct MarketListView: View {
    @State private var store = MarketStore()

    var body: some View {
        List(store.state.markets) { market in
            MarketRow(market: market)
                .onTapGesture {
                    store.dispatch(.selectMarket(market))
                }
        }
        .task {
            await store.loadMarkets()
        }
    }
}
```

### Environment-Based Dependency Injection

```swift
// Define dependencies
protocol MarketServiceProtocol {
    func fetchMarkets() async throws -> [Market]
}

struct MarketService: MarketServiceProtocol {
    func fetchMarkets() async throws -> [Market] {
        // Real implementation
    }
}

struct MockMarketService: MarketServiceProtocol {
    func fetchMarkets() async throws -> [Market] {
        [Market.preview]
    }
}

// Environment key
private struct MarketServiceKey: EnvironmentKey {
    static let defaultValue: any MarketServiceProtocol = MarketService()
}

extension EnvironmentValues {
    var marketService: any MarketServiceProtocol {
        get { self[MarketServiceKey.self] }
        set { self[MarketServiceKey.self] = newValue }
    }
}

// Usage
struct MarketListView: View {
    @Environment(\.marketService) private var marketService
    @State private var markets: [Market] = []

    var body: some View {
        List(markets) { market in
            MarketRow(market: market)
        }
        .task {
            markets = try? await marketService.fetchMarkets() ?? []
        }
    }
}

// Preview with mock
#Preview {
    MarketListView()
        .environment(\.marketService, MockMarketService())
}
```

## Performance Optimization

### Lazy Containers

```swift
// ✅ GOOD: Lazy loading for long lists
struct MarketListView: View {
    let markets: [Market]

    var body: some View {
        ScrollView {
            LazyVStack(spacing: 12) {
                ForEach(markets) { market in
                    MarketCard(market: market)
                }
            }
            .padding()
        }
    }
}

// ❌ BAD: Regular VStack loads all views immediately
struct MarketListView: View {
    let markets: [Market]

    var body: some View {
        ScrollView {
            VStack {  // All 1000 views rendered at once
                ForEach(markets) { market in
                    MarketCard(market: market)
                }
            }
        }
    }
}
```

### Equatable Views

```swift
// ✅ Prevent unnecessary redraws
struct MarketCard: View, Equatable {
    let market: Market

    static func == (lhs: MarketCard, rhs: MarketCard) -> Bool {
        lhs.market.id == rhs.market.id &&
        lhs.market.name == rhs.market.name &&
        lhs.market.status == rhs.market.status
    }

    var body: some View {
        VStack {
            Text(market.name)
            StatusBadge(status: market.status)
        }
    }
}

// Usage with EquatableView
ForEach(markets) { market in
    EquatableView(content: MarketCard(market: market))
}
```

### Computed Properties vs State

```swift
// ✅ GOOD: Computed properties for derived data
struct MarketListView: View {
    let markets: [Market]
    @State private var searchQuery = ""

    var filteredMarkets: [Market] {
        guard !searchQuery.isEmpty else { return markets }
        return markets.filter { $0.name.localizedCaseInsensitiveContains(searchQuery) }
    }

    var body: some View {
        List(filteredMarkets) { market in
            MarketRow(market: market)
        }
        .searchable(text: $searchQuery)
    }
}

// ❌ BAD: Storing derived state (out of sync risk)
struct MarketListView: View {
    let markets: [Market]
    @State private var searchQuery = ""
    @State private var filteredMarkets: [Market] = []  // Redundant state

    var body: some View {
        List(filteredMarkets) { market in
            MarketRow(market: market)
        }
        .onChange(of: searchQuery) {
            filteredMarkets = markets.filter { ... }  // Manual sync required
        }
    }
}
```

## Form Handling Patterns

### Controlled Form with Validation

```swift
struct CreateMarketForm: View {
    @State private var name = ""
    @State private var description = ""
    @State private var endDate = Date()
    @State private var errors: [FieldError] = []

    enum FieldError: Identifiable {
        case nameTooShort
        case nameTooLong
        case descriptionEmpty
        case endDateInPast

        var id: String { String(describing: self) }

        var message: String {
            switch self {
            case .nameTooShort: "Name is required"
            case .nameTooLong: "Name must be under 200 characters"
            case .descriptionEmpty: "Description is required"
            case .endDateInPast: "End date must be in the future"
            }
        }
    }

    private var isValid: Bool {
        errors.isEmpty && !name.isEmpty && !description.isEmpty
    }

    var body: some View {
        Form {
            Section("Market Details") {
                TextField("Name", text: $name)
                if let error = errors.first(where: { $0 == .nameTooShort || $0 == .nameTooLong }) {
                    Text(error.message)
                        .foregroundColor(.red)
                        .font(.caption)
                }

                TextField("Description", text: $description, axis: .vertical)
                    .lineLimit(3...6)

                DatePicker("End Date", selection: $endDate, in: Date()...)
            }

            Button("Create Market") {
                submit()
            }
            .disabled(!isValid)
        }
        .onChange(of: name) { validate() }
        .onChange(of: description) { validate() }
    }

    private func validate() {
        errors = []

        if name.trimmingCharacters(in: .whitespaces).isEmpty {
            errors.append(.nameTooShort)
        } else if name.count > 200 {
            errors.append(.nameTooLong)
        }

        if description.trimmingCharacters(in: .whitespaces).isEmpty {
            errors.append(.descriptionEmpty)
        }

        if endDate <= Date() {
            errors.append(.endDateInPast)
        }
    }

    private func submit() {
        validate()
        guard isValid else { return }
        // Submit form
    }
}
```

## Error Handling Pattern

### Task-Based Error Handling

```swift
struct ContentView: View {
    @State private var data: [Market]?
    @State private var error: Error?
    @State private var isLoading = false

    var body: some View {
        Group {
            if isLoading {
                ProgressView()
            } else if let error {
                ErrorView(error: error) {
                    await loadData()
                }
            } else if let data {
                MarketListView(markets: data)
            }
        }
        .task {
            await loadData()
        }
    }

    private func loadData() async {
        isLoading = true
        error = nil

        do {
            data = try await api.fetchMarkets()
        } catch {
            self.error = error
        }

        isLoading = false
    }
}

struct ErrorView: View {
    let error: Error
    let retry: () async -> Void

    var body: some View {
        VStack(spacing: 16) {
            Image(systemName: "exclamationmark.triangle")
                .font(.largeTitle)
                .foregroundColor(.red)

            Text("Something went wrong")
                .font(.headline)

            Text(error.localizedDescription)
                .font(.caption)
                .foregroundColor(.secondary)

            Button("Try Again") {
                Task {
                    await retry()
                }
            }
            .buttonStyle(.borderedProminent)
        }
        .padding()
    }
}
```

## Animation Patterns

### Implicit Animations

```swift
struct AnimatedMarketList: View {
    let markets: [Market]

    var body: some View {
        List(markets) { market in
            MarketRow(market: market)
                .transition(.asymmetric(
                    insertion: .opacity.combined(with: .move(edge: .trailing)),
                    removal: .opacity.combined(with: .move(edge: .leading))
                ))
        }
        .animation(.spring(response: 0.3, dampingFraction: 0.7), value: markets)
    }
}
```

### Explicit Animations

```swift
struct ExpandableCard: View {
    @State private var isExpanded = false

    var body: some View {
        VStack {
            Button {
                withAnimation(.spring(response: 0.4, dampingFraction: 0.8)) {
                    isExpanded.toggle()
                }
            } label: {
                HStack {
                    Text("Details")
                    Spacer()
                    Image(systemName: "chevron.down")
                        .rotationEffect(.degrees(isExpanded ? 180 : 0))
                }
            }

            if isExpanded {
                Text("Expanded content here")
                    .transition(.opacity.combined(with: .move(edge: .top)))
            }
        }
    }
}
```

### Matched Geometry Effect

```swift
struct MarketGridView: View {
    let markets: [Market]
    @State private var selectedMarket: Market?
    @Namespace private var animation

    var body: some View {
        ZStack {
            ScrollView {
                LazyVGrid(columns: [GridItem(.adaptive(minimum: 150))]) {
                    ForEach(markets) { market in
                        MarketThumbnail(market: market)
                            .matchedGeometryEffect(id: market.id, in: animation)
                            .onTapGesture {
                                withAnimation(.spring(response: 0.4)) {
                                    selectedMarket = market
                                }
                            }
                    }
                }
            }

            if let selected = selectedMarket {
                MarketDetailView(market: selected)
                    .matchedGeometryEffect(id: selected.id, in: animation)
                    .onTapGesture {
                        withAnimation(.spring(response: 0.4)) {
                            selectedMarket = nil
                        }
                    }
            }
        }
    }
}
```

## Accessibility Patterns

### VoiceOver Support

```swift
struct MarketCard: View {
    let market: Market

    var body: some View {
        VStack {
            Text(market.name)
            Text(market.formattedVolume)
            StatusBadge(status: market.status)
        }
        .accessibilityElement(children: .combine)
        .accessibilityLabel("\(market.name), volume \(market.formattedVolume), status \(market.status.rawValue)")
        .accessibilityHint("Double tap to view market details")
        .accessibilityAddTraits(.isButton)
    }
}
```

### Dynamic Type Support

```swift
struct AdaptiveText: View {
    let title: String
    let subtitle: String

    @Environment(\.dynamicTypeSize) private var dynamicTypeSize

    var body: some View {
        if dynamicTypeSize >= .accessibility1 {
            VStack(alignment: .leading) {
                Text(title).font(.headline)
                Text(subtitle).font(.subheadline)
            }
        } else {
            HStack {
                Text(title).font(.headline)
                Spacer()
                Text(subtitle).font(.subheadline)
            }
        }
    }
}
```

### Focus Management

```swift
struct LoginView: View {
    @State private var email = ""
    @State private var password = ""
    @FocusState private var focusedField: Field?

    enum Field {
        case email
        case password
    }

    var body: some View {
        VStack {
            TextField("Email", text: $email)
                .focused($focusedField, equals: .email)
                .submitLabel(.next)
                .onSubmit {
                    focusedField = .password
                }

            SecureField("Password", text: $password)
                .focused($focusedField, equals: .password)
                .submitLabel(.done)
                .onSubmit {
                    submit()
                }
        }
        .onAppear {
            focusedField = .email
        }
    }

    private func submit() {
        focusedField = nil
        // Handle login
    }
}
```

**Remember**: Modern SwiftUI patterns enable maintainable, performant user interfaces. Choose patterns that fit your project complexity.
