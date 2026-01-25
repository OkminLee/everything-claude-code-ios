# iOS Claude Code Plugin

> Claude Code plugin optimized for iOS/Swift development.

[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Plugin-blueviolet.svg)](https://claude.ai/code)

> **Based on [everything-claude-code](https://github.com/affaan-m/everything-claude-code)** by [@affaan-m](https://github.com/affaan-m)
> — An Anthropic hackathon winner's comprehensive Claude Code configuration collection.

---

## Table of Contents

- [Quick Start](#quick-start)
- [Usage Examples](#usage-examples)
- [What's Included](#whats-included)
- [Alternative Installation](#alternative-installation)
- [Configuration](#configuration)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)
- [Acknowledgments](#acknowledgments)
- [한국어 가이드](#한국어-가이드)

---

## Quick Start

### Prerequisites

- [Claude Code CLI](https://claude.ai/code) installed
- macOS with Xcode 15+

### Installation (Recommended)

Run these commands inside Claude Code:

```bash
# 1. Add marketplace
/plugin marketplace add https://github.com/OkminLee/everything-claude-code-ios

# 2. Install plugin
/plugin install everything-claude-code-ios@everything-claude-code-ios
```

### Verify

```
/plugin list
```

You should see `everything-claude-code-ios` in the list. Try `/plan hello` to confirm.

---

## Usage Examples

### Agents

```
@ios-architect Design authentication module architecture
@swift-tdd-guide Write tests for LoginViewModel
@ios-security-reviewer Review Keychain implementation
```

### Commands

```
/plan Implement user profile editing
/swift-review
/xcode-build-fix
/swift-tdd
```

---

## What's Included

### Agents (9)

| Agent | Description |
|-------|-------------|
| `ios-planner` | Feature planning with App Store compliance |
| `ios-architect` | Architecture design (Clean Architecture, TCA) |
| `swift-tdd-guide` | TDD workflow with Swift Testing |
| `swift-code-reviewer` | Code review with Swift best practices |
| `ios-security-reviewer` | Security audit (OWASP, Keychain, ATS) |
| `xcode-build-resolver` | Build error resolution (Swift 6, Privacy Manifest) |
| `xcuitest-runner` | E2E testing with Page Object Model |
| `swift-refactor-cleaner` | Refactoring with Periphery dead code detection |
| `ios-doc-updater` | Documentation with DocC |

### Commands (10)

| Command | Description |
|---------|-------------|
| `/plan` | Create implementation plan |
| `/swift-tdd` | Start TDD session |
| `/swift-review` | Code review current changes |
| `/xcode-build-fix` | Resolve build errors |
| `/xcuitest` | Run E2E tests |
| `/ios-coverage` | Check test coverage |
| `/swift-refactor` | Refactor with dead code detection |
| `/learn` | Learn from codebase patterns |
| `/update-docs` | Update documentation |
| `/update-codemaps` | Update code maps |

### Skills (6)

- `ios-security-review` - Security checklist
- `swift-tdd-workflow` - Red-Green-Refactor guide
- `swift-coding-standards` - Swift best practices
- `swiftui-patterns` - SwiftUI patterns
- `ios-project-guidelines` - Project structure guide
- `backend-patterns` - Backend integration patterns

### Hooks

- **SwiftFormat** - Auto-format Swift files on edit
- **print() warnings** - Alert on debug print statements
- **Pre-push review** - Review before git push
- **Session persistence** - Maintain context across sessions

### MCP Servers

- GitHub - PR/Issue management
- Memory - Persistent context
- Context7 - Documentation lookup

---

## Alternative Installation

### Global Installation

Use across all projects:

```bash
git clone https://github.com/OkminLee/everything-claude-code-ios.git
cp -r everything-claude-code-ios ~/.claude/plugins/everything-claude-code-ios
```

### Project-Local Installation

For project-specific customization:

```bash
cd /path/to/your/ios-project
git clone https://github.com/OkminLee/everything-claude-code-ios.git .claude/plugins/ios
```

### Plugin Command Options

```bash
# Installation scopes
/plugin install everything-claude-code-ios@everything-claude-code-ios --scope user     # All projects (default)
/plugin install everything-claude-code-ios@everything-claude-code-ios --scope project  # Share with team
/plugin install everything-claude-code-ios@everything-claude-code-ios --scope local    # This project only

# Update
/plugin marketplace update everything-claude-code-ios
/plugin update everything-claude-code-ios

# Uninstall
/plugin uninstall everything-claude-code-ios
/plugin marketplace remove everything-claude-code-ios
```

---

## Configuration

### Environment Variables

```bash
export GITHUB_TOKEN="your_github_pat"  # For GitHub MCP server
```

### Customization

Override settings in `.claude/settings.json`:

```json
{
  "plugins": {
    "everything-claude-code-ios": {
      "enabled": true
    }
  }
}
```

---

## Troubleshooting

<details>
<summary><strong>Plugin Not Recognized</strong></summary>

**Symptoms**: Commands like `/plan` don't work

**Solutions**:
1. Verify installation: `/plugin list`
2. For manual installation, check directory exists:
   ```bash
   ls ~/.claude/plugins/everything-claude-code-ios
   ```
3. Check permissions: `chmod -R 755 ~/.claude/plugins/everything-claude-code-ios`
4. Restart Claude Code

</details>

<details>
<summary><strong>MCP Servers Not Connecting</strong></summary>

**Symptoms**: GitHub integration not working

**Solutions**:
1. Verify: `echo $GITHUB_TOKEN`
2. Check token permissions (repo, read:user)
3. Regenerate if expired

</details>

<details>
<summary><strong>Hooks Not Triggering</strong></summary>

**Symptoms**: SwiftFormat doesn't run on save

**Solutions**:
1. Verify hooks.json syntax: `python3 -m json.tool < hooks/hooks.json`
2. Check file pattern matches
3. Ensure `swiftformat` is in PATH: `which swiftformat`

</details>

<details>
<summary><strong>Swift Tools Not Found</strong></summary>

**Solutions**:
```bash
brew install swiftformat
brew install peripheryapp/periphery/periphery
```

</details>

---

## Contributing

Contributions welcome! See [CONTRIBUTING.md](CONTRIBUTING.md).

---

## License

MIT License - see [LICENSE](LICENSE).

---

## Acknowledgments

This plugin is built upon **[everything-claude-code](https://github.com/affaan-m/everything-claude-code)** by [@affaan-m](https://github.com/affaan-m), an Anthropic hackathon winner's production-ready Claude Code configuration.

Thanks to Apple, Swift community, and the Claude Code team.

---

<div align="center">

# 한국어 가이드

</div>

---

## 목차

- [빠른 시작](#빠른-시작)
- [사용 예시](#사용-예시)
- [포함된 구성 요소](#포함된-구성-요소)
- [대체 설치 방법](#대체-설치-방법)
- [설정](#설정)
- [문제 해결](#문제-해결)

---

## 빠른 시작

### 필수 조건

- [Claude Code CLI](https://claude.ai/code) 설치 완료
- macOS 및 Xcode 15+

### 설치 (권장)

Claude Code 내에서 다음 명령어를 실행하세요:

```bash
# 1. 마켓플레이스 추가
/plugin marketplace add https://github.com/OkminLee/everything-claude-code-ios

# 2. 플러그인 설치
/plugin install everything-claude-code-ios@everything-claude-code-ios
```

### 확인

```
/plugin list
```

목록에 `everything-claude-code-ios`가 표시되어야 합니다. `/plan hello`로 동작을 확인하세요.

---

## 사용 예시

### 에이전트

```
@ios-architect 인증 모듈 아키텍처를 설계해줘
@swift-tdd-guide LoginViewModel 테스트를 작성해줘
@ios-security-reviewer Keychain 구현을 검토해줘
```

### 명령어

```
/plan 사용자 프로필 편집 기능 구현
/swift-review
/xcode-build-fix
/swift-tdd
```

---

## 포함된 구성 요소

### 에이전트 (9개)

| 에이전트 | 설명 |
|----------|------|
| `ios-planner` | 기능 기획, App Store 규정 준수 |
| `ios-architect` | 아키텍처 설계 (Clean Architecture, TCA) |
| `swift-tdd-guide` | Swift Testing TDD 가이드 |
| `swift-code-reviewer` | Swift 코드 리뷰 |
| `ios-security-reviewer` | 보안 감사 (OWASP, Keychain, ATS) |
| `xcode-build-resolver` | 빌드 에러 해결 (Swift 6, Privacy Manifest) |
| `xcuitest-runner` | Page Object Model E2E 테스트 |
| `swift-refactor-cleaner` | 리팩토링, 죽은 코드 제거 |
| `ios-doc-updater` | DocC 문서화 |

### 명령어 (10개)

| 명령어 | 설명 |
|--------|------|
| `/plan` | 구현 계획 작성 |
| `/swift-tdd` | TDD 세션 시작 |
| `/swift-review` | 코드 리뷰 |
| `/xcode-build-fix` | 빌드 에러 해결 |
| `/xcuitest` | E2E 테스트 실행 |
| `/ios-coverage` | 테스트 커버리지 확인 |
| `/swift-refactor` | 죽은 코드 제거 리팩토링 |
| `/learn` | 코드베이스 패턴 학습 |
| `/update-docs` | 문서 업데이트 |
| `/update-codemaps` | 코드 맵 업데이트 |

### 스킬 (6개)

- `ios-security-review` - 보안 체크리스트
- `swift-tdd-workflow` - Red-Green-Refactor 가이드
- `swift-coding-standards` - Swift 모범 사례
- `swiftui-patterns` - SwiftUI 패턴
- `ios-project-guidelines` - 프로젝트 구조 가이드
- `backend-patterns` - 백엔드 통합 패턴

### Hooks

- **SwiftFormat** - Swift 파일 편집 시 자동 포맷팅
- **print() 경고** - 디버그 print문 감지
- **푸시 전 리뷰** - git push 전 변경사항 확인
- **세션 지속성** - 세션 간 컨텍스트 유지

### MCP 서버

- GitHub - PR/Issue 관리
- Memory - 컨텍스트 지속성
- Context7 - 문서 조회

---

## 대체 설치 방법

### 글로벌 설치

모든 프로젝트에서 사용:

```bash
git clone https://github.com/OkminLee/everything-claude-code-ios.git
cp -r everything-claude-code-ios ~/.claude/plugins/everything-claude-code-ios
```

### 프로젝트별 설치

프로젝트별 커스터마이징:

```bash
cd /path/to/your/ios-project
git clone https://github.com/OkminLee/everything-claude-code-ios.git .claude/plugins/ios
```

### Plugin 명령어 옵션

```bash
# 설치 범위
/plugin install everything-claude-code-ios@everything-claude-code-ios --scope user     # 모든 프로젝트 (기본값)
/plugin install everything-claude-code-ios@everything-claude-code-ios --scope project  # 팀과 공유
/plugin install everything-claude-code-ios@everything-claude-code-ios --scope local    # 이 프로젝트만

# 업데이트
/plugin marketplace update everything-claude-code-ios
/plugin update everything-claude-code-ios

# 삭제
/plugin uninstall everything-claude-code-ios
/plugin marketplace remove everything-claude-code-ios
```

---

## 설정

### 환경 변수

```bash
export GITHUB_TOKEN="your_github_pat"  # GitHub MCP 서버용
```

### 커스터마이징

`.claude/settings.json`에서 설정 재정의:

```json
{
  "plugins": {
    "everything-claude-code-ios": {
      "enabled": true
    }
  }
}
```

---

## 문제 해결

<details>
<summary><strong>플러그인이 인식되지 않음</strong></summary>

**증상**: `/plan` 같은 명령어가 작동하지 않음

**해결**:
1. 설치 확인: `/plugin list`
2. 수동 설치의 경우 디렉토리 확인:
   ```bash
   ls ~/.claude/plugins/everything-claude-code-ios
   ```
3. 권한 확인: `chmod -R 755 ~/.claude/plugins/everything-claude-code-ios`
4. Claude Code 재시작

</details>

<details>
<summary><strong>MCP 서버 연결 실패</strong></summary>

**증상**: GitHub 연동이 안 됨

**해결**:
1. 확인: `echo $GITHUB_TOKEN`
2. 토큰 권한 확인 (repo, read:user)
3. 만료 시 재발급

</details>

<details>
<summary><strong>Hooks가 동작하지 않음</strong></summary>

**증상**: SwiftFormat이 저장 시 실행되지 않음

**해결**:
1. hooks.json 문법 확인: `python3 -m json.tool < hooks/hooks.json`
2. 파일 패턴 일치 여부 확인
3. `swiftformat` PATH 확인: `which swiftformat`

</details>

<details>
<summary><strong>Swift 도구를 찾을 수 없음</strong></summary>

**해결**:
```bash
brew install swiftformat
brew install peripheryapp/periphery/periphery
```

</details>

---

## 감사의 말

이 플러그인은 [@affaan-m](https://github.com/affaan-m)이 만든 **[everything-claude-code](https://github.com/affaan-m/everything-claude-code)**를 기반으로 합니다. Anthropic 해커톤 수상작으로, 프로덕션 수준의 Claude Code 설정을 제공합니다.

Apple, Swift 커뮤니티, Claude Code 팀에 감사드립니다.
