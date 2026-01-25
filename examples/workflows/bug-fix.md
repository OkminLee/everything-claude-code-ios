# 버그 수정 워크플로우

버그를 분석하고 수정하는 플로우입니다.

---

## 개요

```
Reproduce → Analyze → Test First → Fix → Verify → Review
```

---

## Phase 1: 버그 재현

### 1.1 버그 리포트 분석

```
버그: 로그인 후 간헐적으로 앱이 크래시됨
- 환경: iOS 17.2, iPhone 15 Pro
- 재현율: ~30%
- 크래시 로그: EXC_BAD_ACCESS
```

### 1.2 재현 조건 확인

```bash
# 크래시 로그 확인
xcrun simctl spawn booted log stream --predicate 'process == "MyApp"'
```

---

## Phase 2: 원인 분석

### 2.1 코드 분석

```
@swift-code-reviewer
로그인 후 크래시가 발생합니다. 다음 파일들을 분석해주세요:
- AuthService.swift
- HomeViewModel.swift
- AppCoordinator.swift
```

### 2.2 분석 결과 예시

```markdown
## 원인 분석

**Root Cause**: Race Condition

`HomeViewModel`이 `AuthService`의 로그인 완료를 기다리지 않고
사용자 정보에 접근하여 크래시 발생.

**문제 코드**:
```swift
// HomeViewModel.swift:45
func onAppear() {
    Task {
        // authService.currentUser가 아직 nil일 수 있음
        let user = authService.currentUser!  // ❌ Force unwrap
        await loadDashboard(for: user)
    }
}
```
```

---

## Phase 3: 테스트 먼저 작성

### 3.1 실패하는 테스트 작성

```
@swift-tdd-guide
HomeViewModel에서 로그인 완료 전 접근 시 크래시되는 버그가 있습니다.
이 버그를 재현하는 테스트를 먼저 작성해주세요.
```

```swift
// Tests/HomeViewModelTests.swift
@Test("로그인 완료 전 onAppear 호출 시 크래시하지 않음")
func onAppear_beforeLoginComplete_doesNotCrash() async {
    // Given
    mockAuthService.currentUser = nil  // 로그인 미완료 상태

    // When
    await sut.onAppear()

    // Then
    #expect(sut.state == .loading)  // 크래시 없이 로딩 상태
}

@Test("로그인 완료 후 대시보드 정상 로드")
func onAppear_afterLoginComplete_loadsDashboard() async {
    // Given
    mockAuthService.currentUser = .mock

    // When
    await sut.onAppear()

    // Then
    #expect(sut.dashboardItems.isEmpty == false)
}
```

### 3.2 테스트 실행 (실패 확인)

```bash
swift test --filter HomeViewModelTests
# ❌ Test failed: Crash on force unwrap
```

---

## Phase 4: 버그 수정

### 4.1 수정 코드

```swift
// HomeViewModel.swift:45
func onAppear() {
    Task {
        // ✅ 안전한 옵셔널 처리
        guard let user = authService.currentUser else {
            state = .loading
            await waitForAuthentication()
            return
        }
        await loadDashboard(for: user)
    }
}

private func waitForAuthentication() async {
    // 인증 완료 대기 로직
    for await user in authService.userStream {
        if let user {
            await loadDashboard(for: user)
            break
        }
    }
}
```

### 4.2 테스트 재실행

```bash
swift test --filter HomeViewModelTests
# ✅ All tests passed
```

---

## Phase 5: 검증

### 5.1 회귀 테스트

```bash
# 전체 테스트 실행
swift test

# UI 테스트 실행
xcodebuild test -scheme MyApp -destination 'platform=iOS Simulator,name=iPhone 15'
```

### 5.2 수동 테스트

- [ ] 로그인 직후 홈 화면 진입 (10회 반복)
- [ ] 네트워크 느린 상황에서 테스트
- [ ] 백그라운드 → 포그라운드 전환

---

## Phase 6: 리뷰 및 머지

### 6.1 코드 리뷰

```
@swift-code-reviewer
로그인 후 크래시 버그 수정 PR을 리뷰해주세요.
```

### 6.2 커밋

```bash
git commit -m "fix: Crash after login due to race condition

- Add guard for nil currentUser
- Implement waitForAuthentication for async login flow
- Add regression tests

Fixes #123

Co-Authored-By: Claude <noreply@anthropic.com>"
```

---

## 체크리스트

### 버그 수정 완료 조건

- [ ] 버그 재현 테스트 작성
- [ ] 수정 후 테스트 통과
- [ ] 기존 테스트 모두 통과
- [ ] 수동 테스트 완료
- [ ] 코드 리뷰 승인
- [ ] 관련 이슈 링크

### 사용 에이전트

1. `@swift-code-reviewer` - 원인 분석
2. `@swift-tdd-guide` - 회귀 테스트 작성
3. `@xcode-build-resolver` - 빌드 에러 발생 시
