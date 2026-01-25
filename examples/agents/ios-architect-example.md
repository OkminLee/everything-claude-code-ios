# ios-architect 사용 예제

새 프로젝트 아키텍처 설계 세션 예시입니다.

---

## 시나리오

신규 iOS 앱 프로젝트의 아키텍처를 설계하는 상황입니다.

## 세션 예시

### 사용자 요청

```
@ios-architect
새로운 이커머스 앱을 개발하려고 합니다.
- iOS 17+ 타겟
- 2-3명 규모 팀
- 오프라인 지원 필요
- 결제 기능 포함

아키텍처를 설계해주세요.
```

### 에이전트 응답 예시

```markdown
## 1. 아키텍처 분석

### 요구사항 정리
- **팀 규모**: 2-3명 (중소규모)
- **iOS 버전**: 17+ (최신 API 활용 가능)
- **핵심 기능**: 오프라인 지원, 결제

### 권장 아키텍처: Clean Architecture + MVVM

팀 규모와 프로젝트 복잡도를 고려할 때, TCA보다 MVVM이 적합합니다.
- TCA는 학습 곡선이 높고 보일러플레이트가 많음
- MVVM은 팀원들이 쉽게 적응 가능

## 2. 프로젝트 구조

```
EcommerceApp/
├── App/
│   ├── EcommerceApp.swift       # @main
│   └── AppDelegate.swift
├── Core/
│   ├── DI/
│   │   └── Container.swift      # 의존성 컨테이너
│   ├── Extensions/
│   └── Utilities/
├── Domain/
│   ├── Entities/
│   │   ├── Product.swift
│   │   ├── Cart.swift
│   │   └── Order.swift
│   ├── UseCases/
│   │   ├── ProductUseCase.swift
│   │   └── OrderUseCase.swift
│   └── Repositories/            # 프로토콜만
│       ├── ProductRepositoryProtocol.swift
│       └── OrderRepositoryProtocol.swift
├── Data/
│   ├── Repositories/            # 구현체
│   │   ├── ProductRepository.swift
│   │   └── OrderRepository.swift
│   ├── Network/
│   │   ├── APIClient.swift
│   │   └── Endpoints/
│   ├── Persistence/
│   │   ├── SwiftDataContainer.swift
│   │   └── Models/
│   └── Mappers/
├── Features/
│   ├── Home/
│   │   ├── HomeView.swift
│   │   └── HomeViewModel.swift
│   ├── ProductDetail/
│   ├── Cart/
│   ├── Checkout/
│   └── Profile/
└── Resources/
```

## 3. 핵심 설계 결정

### 오프라인 지원
- **SwiftData** 사용 (iOS 17+ 전용)
- Repository 패턴으로 로컬/원격 데이터 소스 추상화
- 동기화 전략: 온라인 복귀 시 자동 동기화

### 결제
- **StoreKit 2** 사용
- 서버 사이드 영수증 검증 필수
- 별도 PaymentService로 분리

### 상태 관리
- **@Observable** 사용 (iOS 17+)
- @ObservableObject 대비 보일러플레이트 감소
- fine-grained 업데이트로 성능 향상

## 4. 의존성

```swift
// Package.swift
dependencies: [
    // 네트워크
    .package(url: "https://github.com/Alamofire/Alamofire.git", from: "5.8.0"),
    // 이미지
    .package(url: "https://github.com/onevcat/Kingfisher.git", from: "7.10.0"),
    // 린팅
    .package(url: "https://github.com/nicklockwood/SwiftFormat.git", from: "0.53.0"),
]
```

## 5. 다음 단계

1. 프로젝트 생성 및 폴더 구조 설정
2. Core/DI 컨테이너 구현
3. Domain 레이어 Entity 정의
4. Data 레이어 Repository 구현
5. 첫 번째 Feature (Home) TDD로 개발
```

---

## 핵심 포인트

1. **팀 규모에 맞는 아키텍처 선택**: 소규모 팀에서는 MVVM이 TCA보다 적합
2. **iOS 버전에 따른 기술 스택**: iOS 17+라면 @Observable, SwiftData 활용
3. **단계별 구현 계획**: 구조 설정 → Domain → Data → Features 순서
4. **의존성 최소화**: 필수 라이브러리만 추가

## 관련 에이전트

- `@ios-planner` - 기능 단위 계획 수립
- `@swift-tdd-guide` - TDD 방식으로 구현 시작
- `@ios-security-reviewer` - 결제 관련 보안 검토
