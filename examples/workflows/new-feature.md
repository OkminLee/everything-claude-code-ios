# 신규 기능 개발 워크플로우

새로운 기능을 처음부터 끝까지 개발하는 전체 플로우입니다.

---

## 개요

```
Plan → Architect → TDD → Implement → Review → Security → Merge
```

---

## Phase 1: 계획 수립

### 1.1 기능 계획

```
@ios-planner
사용자 프로필 편집 기능을 개발하려고 합니다.
- 이름, 프로필 사진, 자기소개 수정
- 사진은 카메라/갤러리에서 선택
- 변경사항 서버 동기화
```

**결과물:**
- 기능 요구사항 문서
- Red Flags 체크 (App Store 규정)
- 예상 작업 목록

### 1.2 아키텍처 설계

```
@ios-architect
프로필 편집 기능의 아키텍처를 설계해주세요.
기존 프로젝트는 Clean Architecture + MVVM 사용 중입니다.
```

**결과물:**
- 모듈 구조
- 데이터 흐름
- 의존성 정의

---

## Phase 2: TDD 개발

### 2.1 도메인 레이어 TDD

```
@swift-tdd-guide
UpdateProfileUseCase를 TDD로 개발해주세요.
- 이름 업데이트
- 프로필 사진 업데이트
- 유효성 검증 포함
```

**순서:**
1. 테스트 케이스 작성
2. UseCase 구현
3. Repository Protocol 정의

### 2.2 데이터 레이어 구현

```swift
// Tests에서 정의한 Protocol 기반으로 구현
final class ProfileRepository: ProfileRepositoryProtocol {
    private let apiClient: APIClientProtocol
    private let imageUploader: ImageUploaderProtocol

    func updateProfile(_ profile: Profile) async throws -> Profile {
        // 구현
    }
}
```

### 2.3 Presentation 레이어

```
@swift-tdd-guide
ProfileEditViewModel을 TDD로 개발해주세요.
- 입력 유효성 검증
- 로딩 상태 관리
- 에러 핸들링
```

---

## Phase 3: UI 구현

### 3.1 SwiftUI View 작성

```swift
struct ProfileEditView: View {
    @State private var viewModel: ProfileEditViewModel

    var body: some View {
        Form {
            Section("기본 정보") {
                TextField("이름", text: $viewModel.name)
                // ...
            }
        }
        .navigationTitle("프로필 편집")
    }
}
```

### 3.2 UI 테스트 작성

```
@xcuitest-runner
프로필 편집 화면의 E2E 테스트를 작성해주세요.
- 프로필 수정 후 저장
- 유효하지 않은 입력 시 에러 표시
- 사진 선택 플로우
```

---

## Phase 4: 리뷰

### 4.1 코드 리뷰

```
@swift-code-reviewer
프로필 편집 기능 코드를 리뷰해주세요.
```

**체크포인트:**
- [ ] 메모리 관리 (weak self)
- [ ] 에러 핸들링
- [ ] 접근성

### 4.2 보안 리뷰

```
@ios-security-reviewer
프로필 편집 기능의 보안을 검토해주세요.
- 이미지 업로드
- 사용자 데이터 처리
```

**체크포인트:**
- [ ] 이미지 메타데이터 제거
- [ ] 입력 검증
- [ ] 네트워크 보안

---

## Phase 5: 완료

### 5.1 문서 업데이트

```
@ios-doc-updater
프로필 편집 기능 추가에 따른 문서를 업데이트해주세요.
```

### 5.2 PR 생성

```bash
git checkout -b feature/profile-edit
git add .
git commit -m "feat: Add profile edit feature

- Add ProfileEditView with form UI
- Implement UpdateProfileUseCase with validation
- Add unit tests (95% coverage)
- Add E2E tests for critical flows

Co-Authored-By: Claude <noreply@anthropic.com>"

git push -u origin feature/profile-edit
gh pr create --title "feat: Add profile edit feature" --body "..."
```

---

## 체크리스트

### 개발 완료 조건

- [ ] 모든 유닛 테스트 통과
- [ ] UI 테스트 통과
- [ ] 코드 리뷰 승인
- [ ] 보안 리뷰 통과
- [ ] 80%+ 커버리지
- [ ] 접근성 검증
- [ ] 다크모드 확인

### 에이전트 사용 순서

1. `@ios-planner` - 요구사항 정리
2. `@ios-architect` - 아키텍처 설계
3. `@swift-tdd-guide` - TDD 개발
4. `@xcuitest-runner` - E2E 테스트
5. `@swift-code-reviewer` - 코드 리뷰
6. `@ios-security-reviewer` - 보안 검토
7. `@ios-doc-updater` - 문서 업데이트
