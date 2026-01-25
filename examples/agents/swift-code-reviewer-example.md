# swift-code-reviewer 사용 예제

PR 코드 리뷰 세션 예시입니다.

---

## 시나리오

새 기능 PR이 올라왔고, 코드 리뷰를 요청하는 상황입니다.

## 세션 예시

### 사용자 요청

```
@swift-code-reviewer
방금 올린 PR 리뷰해줘. 사용자 인증 기능 추가했어.
```

### 에이전트 응답 예시

```markdown
## Code Review Summary

### Files Reviewed
- `Sources/Features/Auth/LoginView.swift` (New)
- `Sources/Features/Auth/LoginViewModel.swift` (New)
- `Sources/Services/AuthService.swift` (New)
- `Sources/Core/Keychain/KeychainManager.swift` (Modified)

---

### 🔴 Critical Issues (2)

| Issue | Location | Fix |
|-------|----------|-----|
| 토큰이 UserDefaults에 저장됨 | AuthService.swift:42 | Keychain 사용 |
| 하드코딩된 API 키 | AuthService.swift:15 | 환경 변수로 이동 |

**1. 토큰이 UserDefaults에 저장됨**
```swift
// ❌ AuthService.swift:42
UserDefaults.standard.set(token, forKey: "authToken")

// ✅ 수정
try KeychainManager.shared.save(
    token,
    forKey: "authToken",
    accessibility: .whenUnlockedThisDeviceOnly
)
```

**2. 하드코딩된 API 키**
```swift
// ❌ AuthService.swift:15
private let apiKey = "sk-proj-xxxxx"

// ✅ 수정 - xcconfig 또는 환경 변수 사용
private var apiKey: String {
    Bundle.main.infoDictionary?["API_KEY"] as? String ?? ""
}
```

---

### 🟡 High Issues (3)

| Issue | Location | Fix |
|-------|----------|-----|
| weak self 누락 | LoginViewModel.swift:28 | [weak self] 추가 |
| Force unwrap 사용 | LoginView.swift:55 | guard let 사용 |
| print() 문 존재 | AuthService.swift:67 | #if DEBUG로 감싸기 |

**1. weak self 누락**
```swift
// ❌ LoginViewModel.swift:28
Task {
    let result = await authService.login(email: email, password: password)
    self.handleResult(result)  // 잠재적 retain cycle
}

// ✅ 수정
Task { [weak self] in
    guard let self else { return }
    let result = await authService.login(email: email, password: password)
    self.handleResult(result)
}
```

---

### 🟢 Suggestions (2)

1. **LoginViewModel을 @MainActor로 마킹** - UI 업데이트 안전성
2. **에러 타입 세분화** - AuthError enum 정의 권장

---

### Good Practices Found

- Dependency Injection 패턴 잘 적용됨
- Protocol 기반 추상화로 테스트 용이
- SwiftUI 뷰가 200줄 이하로 적절한 크기

---

### Verdict: ❌ Block

Critical 이슈 2개 해결 후 재리뷰 필요합니다.

**우선순위:**
1. UserDefaults → Keychain 마이그레이션
2. 하드코딩된 API 키 제거
3. weak self 추가
```

---

## 핵심 포인트

1. **보안 이슈 최우선**: UserDefaults 토큰 저장, 하드코딩된 시크릿은 Critical
2. **메모리 관리**: Task 내 weak self 패턴 확인
3. **심각도 분류**: Critical → High → Suggestions 순서
4. **구체적인 수정 코드 제공**: 문제점과 해결책을 코드로 명시

## 관련 에이전트

- `@ios-security-reviewer` - 보안 이슈 심층 분석
- `@swift-refactor-cleaner` - 코드 정리 및 리팩토링
