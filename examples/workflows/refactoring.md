# 리팩토링 워크플로우

코드 품질 개선 및 기술 부채 해소 플로우입니다.

---

## 개요

```
Analyze → Plan → Test Coverage → Refactor → Verify → Clean Up
```

---

## Phase 1: 현황 분석

### 1.1 데드 코드 분석

```
@swift-refactor-cleaner
현재 프로젝트의 데드 코드와 미사용 의존성을 분석해주세요.
```

**분석 결과 예시:**
```markdown
## 분석 결과

### 미사용 코드
- `OldNetworkManager.swift` - 전체 파일 미사용
- `LegacyUserModel.swift` - 새 모델로 대체됨
- `Utils/DateHelper.swift:45-78` - 미사용 함수 3개

### 미사용 의존성 (SPM)
- `SwiftyJSON` - Codable로 대체됨
- `PromiseKit` - async/await로 마이그레이션 완료

### 중복 코드
- `PrimaryButton.swift` + `ActionButton.swift` → 통합 필요
```

### 1.2 아키텍처 분석

```
@ios-architect
현재 코드베이스의 아키텍처 문제점을 분석해주세요.
```

**분석 결과 예시:**
```markdown
## 아키텍처 이슈

### 레이어 위반
- `HomeView.swift:89` - View에서 직접 API 호출
- `ProductService.swift:34` - Service에서 UI 코드 참조

### 개선 필요
- ViewModel들이 Singleton 의존 → DI 컨테이너 도입 필요
- 에러 핸들링 일관성 부족
```

---

## Phase 2: 리팩토링 계획

### 2.1 우선순위 결정

| 순위 | 작업 | 위험도 | 영향 범위 |
|------|------|--------|----------|
| 1 | 데드 코드 제거 | 낮음 | 없음 |
| 2 | 미사용 의존성 제거 | 낮음 | 빌드 |
| 3 | 중복 코드 통합 | 중간 | UI |
| 4 | DI 컨테이너 도입 | 높음 | 전체 |

### 2.2 단계별 계획

```
Week 1: 데드 코드 + 미사용 의존성 제거
Week 2: 중복 코드 통합
Week 3-4: DI 컨테이너 도입
```

---

## Phase 3: 테스트 커버리지 확보

리팩토링 전 테스트 커버리지를 확보합니다.

### 3.1 현재 커버리지 확인

```bash
xcodebuild test \
  -scheme MyApp \
  -destination 'platform=iOS Simulator,name=iPhone 15' \
  -enableCodeCoverage YES \
  -resultBundlePath TestResults.xcresult

xcrun xccov view --report TestResults.xcresult
# Current Coverage: 65%
```

### 3.2 핵심 로직 테스트 보강

```
@swift-tdd-guide
리팩토링 전 HomeViewModel의 테스트 커버리지를 80% 이상으로 올려주세요.
```

---

## Phase 4: 리팩토링 실행

### 4.1 데드 코드 제거

```
@swift-refactor-cleaner
분석된 데드 코드를 안전하게 제거해주세요.
DELETION_LOG.md에 기록해주세요.
```

**결과:**
```markdown
# DELETION_LOG.md

## 2024-01-15 Refactor Session

### 삭제된 파일
- OldNetworkManager.swift - NetworkService로 대체됨
- LegacyUserModel.swift - User.swift로 통합됨

### 삭제된 코드
- Utils/DateHelper.swift:45-78 (33줄) - 미사용 함수

### 제거된 의존성
- SwiftyJSON (Package.swift에서 제거)
- PromiseKit (Package.swift에서 제거)

### 영향
- 파일 3개 삭제
- 코드 150줄 감소
- 의존성 2개 제거
```

### 4.2 중복 코드 통합

```swift
// Before: 2개 파일
// PrimaryButton.swift
// ActionButton.swift

// After: 1개 파일로 통합
struct AppButton: View {
    enum Style {
        case primary
        case secondary
        case destructive
    }

    let title: String
    let style: Style
    let action: () -> Void

    var body: some View {
        Button(title, action: action)
            .buttonStyle(AppButtonStyle(style: style))
    }
}
```

### 4.3 DI 컨테이너 도입

```swift
// Before: Singleton
let service = NetworkService.shared

// After: DI Container
@Observable
final class AppContainer {
    let networkService: NetworkServiceProtocol
    let authService: AuthServiceProtocol
    // ...

    init() {
        self.networkService = NetworkService()
        self.authService = AuthService(network: networkService)
    }
}

// Usage
struct MyApp: App {
    @State private var container = AppContainer()

    var body: some Scene {
        WindowGroup {
            ContentView()
                .environment(container)
        }
    }
}
```

---

## Phase 5: 검증

### 5.1 테스트 실행

```bash
# 모든 테스트 통과 확인
swift test

# 커버리지 유지 확인
xcrun xccov view --report TestResults.xcresult
# Coverage: 82% (개선됨)
```

### 5.2 빌드 검증

```
@xcode-build-resolver
리팩토링 후 빌드 에러가 있으면 수정해주세요.
```

### 5.3 성능 검증

```bash
# 빌드 시간 확인
xcodebuild build -scheme MyApp -destination 'generic/platform=iOS' \
  -buildTimeSummary

# Before: 45s
# After: 38s (15% 개선)
```

---

## Phase 6: 문서화

### 6.1 변경 사항 기록

```
@ios-doc-updater
리팩토링으로 변경된 아키텍처를 문서에 반영해주세요.
```

### 6.2 PR 생성

```bash
git commit -m "refactor: Remove dead code and introduce DI container

- Remove unused files (OldNetworkManager, LegacyUserModel)
- Remove unused dependencies (SwiftyJSON, PromiseKit)
- Consolidate duplicate buttons into AppButton
- Introduce AppContainer for dependency injection
- See DELETION_LOG.md for details

Breaking Changes:
- NetworkService.shared removed, use AppContainer instead

Co-Authored-By: Claude <noreply@anthropic.com>"
```

---

## 체크리스트

### 리팩토링 완료 조건

- [ ] 모든 테스트 통과
- [ ] 커버리지 유지 또는 개선
- [ ] DELETION_LOG.md 업데이트
- [ ] 아키텍처 문서 업데이트
- [ ] 빌드 시간 유지 또는 개선
- [ ] 코드 리뷰 승인

### 사용 에이전트

1. `@swift-refactor-cleaner` - 데드 코드 분석/제거
2. `@ios-architect` - 아키텍처 개선
3. `@swift-tdd-guide` - 테스트 커버리지 확보
4. `@xcode-build-resolver` - 빌드 에러 수정
5. `@ios-doc-updater` - 문서 업데이트
