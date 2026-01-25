# swift-tdd-guide 사용 예제

TDD 방식으로 새 기능을 개발하는 세션 예시입니다.

---

## 시나리오

상품 검색 기능을 TDD로 개발하는 상황입니다.

## 세션 예시

### 사용자 요청

```
@swift-tdd-guide
상품 검색 기능을 TDD로 개발하고 싶어.
- 검색어로 상품 목록 조회
- 빈 검색어는 전체 목록 반환
- 네트워크 에러 시 로컬 캐시 반환
```

### 에이전트 응답 예시

```markdown
## TDD 워크플로우 시작

### Step 1: 테스트 케이스 정의

```swift
// Tests/ProductSearchServiceTests.swift
import Testing
@testable import MyApp

@Suite("ProductSearchService Tests")
struct ProductSearchServiceTests {

    var sut: ProductSearchService!
    var mockRepository: MockProductRepository!

    init() {
        mockRepository = MockProductRepository()
        sut = ProductSearchService(repository: mockRepository)
    }

    // MARK: - 검색 성공 케이스

    @Test("검색어로 상품 조회 시 일치하는 상품 반환")
    func searchWithQuery_returnsMatchingProducts() async throws {
        // Given
        let expectedProducts = [Product.mock(name: "iPhone")]
        mockRepository.searchResult = .success(expectedProducts)

        // When
        let result = try await sut.search(query: "iPhone")

        // Then
        #expect(result.count == 1)
        #expect(result.first?.name == "iPhone")
    }

    @Test("빈 검색어로 조회 시 전체 상품 반환")
    func searchWithEmptyQuery_returnsAllProducts() async throws {
        // Given
        let allProducts = [Product.mock(), Product.mock()]
        mockRepository.allProductsResult = .success(allProducts)

        // When
        let result = try await sut.search(query: "")

        // Then
        #expect(result.count == 2)
    }

    // MARK: - 에러 케이스

    @Test("네트워크 에러 시 캐시된 결과 반환")
    func searchWithNetworkError_returnsCachedResults() async throws {
        // Given
        mockRepository.searchResult = .failure(NetworkError.noConnection)
        mockRepository.cachedProducts = [Product.mock(name: "Cached")]

        // When
        let result = try await sut.search(query: "test")

        // Then
        #expect(result.first?.name == "Cached")
    }

    @Test("네트워크 에러이고 캐시도 없으면 에러 throw")
    func searchWithNetworkErrorAndNoCache_throwsError() async {
        // Given
        mockRepository.searchResult = .failure(NetworkError.noConnection)
        mockRepository.cachedProducts = []

        // When/Then
        await #expect(throws: NetworkError.self) {
            try await sut.search(query: "test")
        }
    }
}
```

### Step 2: 테스트 실행 (RED)

```bash
swift test --filter ProductSearchServiceTests
```

예상 결과: 컴파일 에러 (ProductSearchService가 아직 없음)

### Step 3: 최소 구현 (GREEN)

```swift
// Sources/Services/ProductSearchService.swift
import Foundation

protocol ProductSearchServiceProtocol: Sendable {
    func search(query: String) async throws -> [Product]
}

final class ProductSearchService: ProductSearchServiceProtocol {
    private let repository: ProductRepositoryProtocol

    init(repository: ProductRepositoryProtocol) {
        self.repository = repository
    }

    func search(query: String) async throws -> [Product] {
        // 빈 검색어: 전체 반환
        if query.isEmpty {
            return try await repository.getAllProducts()
        }

        // 검색 시도
        do {
            return try await repository.searchProducts(query: query)
        } catch {
            // 네트워크 에러: 캐시 반환
            let cached = repository.getCachedProducts(for: query)
            if cached.isEmpty {
                throw error
            }
            return cached
        }
    }
}
```

### Step 4: Mock 구현

```swift
// Tests/Mocks/MockProductRepository.swift
final class MockProductRepository: ProductRepositoryProtocol, @unchecked Sendable {
    var searchResult: Result<[Product], Error> = .success([])
    var allProductsResult: Result<[Product], Error> = .success([])
    var cachedProducts: [Product] = []

    func searchProducts(query: String) async throws -> [Product] {
        try searchResult.get()
    }

    func getAllProducts() async throws -> [Product] {
        try allProductsResult.get()
    }

    func getCachedProducts(for query: String) -> [Product] {
        cachedProducts
    }
}
```

### Step 5: 테스트 재실행

```bash
swift test --filter ProductSearchServiceTests
# ✅ All tests passed
```

### Step 6: 리팩토링

```swift
// 에러 핸들링 개선
func search(query: String) async throws -> [Product] {
    guard !query.isEmpty else {
        return try await repository.getAllProducts()
    }

    return try await fetchWithFallback(query: query)
}

private func fetchWithFallback(query: String) async throws -> [Product] {
    do {
        return try await repository.searchProducts(query: query)
    } catch let error as NetworkError {
        return try handleNetworkError(error, query: query)
    }
}

private func handleNetworkError(_ error: NetworkError, query: String) throws -> [Product] {
    let cached = repository.getCachedProducts(for: query)
    guard !cached.isEmpty else { throw error }
    return cached
}
```

### Step 7: 커버리지 확인

```bash
swift test --enable-code-coverage
# Coverage: 95%
```

## 완료!

Red → Green → Refactor 사이클을 완료했습니다.
```

---

## 핵심 포인트

1. **테스트 먼저 작성**: 구현 전에 모든 테스트 케이스 정의
2. **Swift Testing 문법**: `@Test`, `#expect`, `@Suite` 사용
3. **Mock 패턴**: Protocol 기반 의존성 주입으로 테스트 가능
4. **단계별 진행**: RED → GREEN → REFACTOR

## 관련 에이전트

- `@xcuitest-runner` - UI 테스트 추가
- `@swift-code-reviewer` - 구현 코드 리뷰
