# iOS Claude Code Plugin

> Claude Code plugin optimized for iOS/Swift development.

[![Swift](https://img.shields.io/badge/Swift-5.9+-orange.svg)](https://swift.org)
[![Platform](https://img.shields.io/badge/Platform-iOS%2017+-blue.svg)](https://developer.apple.com/ios/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Plugin-blueviolet.svg)](https://code.claude.com)

---

## Installation

### Option 1: Plugin Marketplace (Recommended)

```bash
# Add this repository as a marketplace
claude plugin marketplace add https://github.com/mininny/everything-claude-code-ios

# Install the plugin
claude plugin install everything-claude-code-ios
```

### Option 2: Direct Installation

```bash
# Clone and install directly
git clone https://github.com/mininny/everything-claude-code-ios.git
cd everything-claude-code-ios

# Copy to Claude Code plugin directory
cp -r . ~/.claude/plugins/everything-claude-code-ios/
```

### Option 3: Project-Local Installation

```bash
# For project-specific use
git clone https://github.com/mininny/everything-claude-code-ios.git .claude/plugins/ios
```

---

## What's Included

### Agents (9)

Specialized AI agents for iOS development tasks:

| Agent | Description |
|-------|-------------|
| `ios-planner` | Feature planning with App Store compliance checks |
| `ios-architect` | Architecture design (Clean Architecture, TCA) |
| `swift-tdd-guide` | TDD workflow with Swift Testing framework |
| `swift-code-reviewer` | Code review with Swift best practices |
| `ios-security-reviewer` | Security audit (OWASP, Keychain, ATS) |
| `xcode-build-resolver` | Build error resolution (Swift 6, Privacy Manifest) |
| `xcuitest-runner` | E2E testing with Page Object Model |
| `swift-refactor-cleaner` | Refactoring with Periphery dead code detection |
| `ios-doc-updater` | Documentation with DocC and Codemaps |

### Skills (8+)

Reusable capabilities Claude can invoke:

- `/ios-security-review` - Comprehensive security checklist
- `/swift-tdd-workflow` - Red-Green-Refactor cycle guide
- Swift coding standards, SwiftUI patterns, and more

### Commands (10)

Slash commands for common workflows:

- `/plan` - Create implementation plan before coding
- `/swift-tdd` - Start TDD session
- `/swift-review` - Code review current changes
- `/xcode-build-fix` - Resolve build errors
- `/xcuitest` - Run E2E tests
- `/ios-coverage` - Check test coverage
- And more...

### Hooks

Automated behaviors for iOS development:

- **Pre-push review** - Pause before git push for final review
- **SwiftFormat** - Auto-format Swift files on edit
- **print() warnings** - Alert on debug print statements
- **Session persistence** - Maintain context across sessions

### MCP Servers

Pre-configured Model Context Protocol servers:

- GitHub - PR/Issue management
- Memory - Persistent context
- Sequential-thinking - Complex reasoning
- Context7 - Documentation lookup

---

## Usage Examples

### Using Agents

```
@ios-architect Design the architecture for a new authentication module
@swift-tdd-guide Help me write tests for the LoginViewModel
@ios-security-reviewer Review the Keychain implementation
```

### Using Commands

```
/plan Implement user profile editing feature
/swift-review
/xcode-build-fix
```

---

## Architecture Support

### Clean Architecture

```
Presentation Layer (SwiftUI Views, ViewModels)
    ↓
Domain Layer (Use Cases, Entities, Repository Protocols)
    ↓
Data Layer (Repository Implementations, Data Sources)
    ↓
Infrastructure Layer (Network, Database, Keychain)
```

### TCA (The Composable Architecture)

```
Feature
├── State
├── Action
├── Reducer
├── Environment/Dependencies
└── View
```

---

## Testing Strategy

| Test Type | Framework | Purpose |
|-----------|-----------|---------|
| Unit Tests | Swift Testing | Business logic, ViewModels, Use Cases |
| Integration Tests | Swift Testing | Repository + Data Source integration |
| UI Tests | XCUITest | User flows, accessibility |
| Snapshot Tests | swift-snapshot-testing | UI regression |
| Performance Tests | XCTest | Memory, CPU profiling |

---

## Configuration

### Environment Variables

For MCP servers, set these environment variables:

```bash
export GITHUB_TOKEN="your_github_pat"
```

### Customization

Override plugin settings in your project's `.claude/settings.json`:

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

## Contributing

Contributions are welcome! Please read [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

---

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## Acknowledgments

- Based on [everything-claude-code](https://github.com/anthropics/everything-claude-code)
- Inspired by iOS development best practices from Apple and the Swift community

---

# 한국어 가이드

## 설치 방법

### 방법 1: 플러그인 마켓플레이스 (권장)

```bash
# 마켓플레이스 추가
claude plugin marketplace add https://github.com/mininny/everything-claude-code-ios

# 플러그인 설치
claude plugin install everything-claude-code-ios
```

### 방법 2: 직접 설치

```bash
# 저장소 클론 후 플러그인 디렉토리에 복사
git clone https://github.com/mininny/everything-claude-code-ios.git
cp -r everything-claude-code-ios ~/.claude/plugins/
```

### 방법 3: 프로젝트별 설치

```bash
# 특정 프로젝트에서만 사용
git clone https://github.com/mininny/everything-claude-code-ios.git .claude/plugins/ios
```

---

## 포함된 구성 요소

### 에이전트 (9개)

| 에이전트 | 설명 |
|----------|------|
| `ios-planner` | 기능 기획, App Store 규정 준수 확인 |
| `ios-architect` | 아키텍처 설계 (Clean Architecture, TCA) |
| `swift-tdd-guide` | Swift Testing 프레임워크 TDD 가이드 |
| `swift-code-reviewer` | Swift 코드 리뷰 |
| `ios-security-reviewer` | 보안 감사 (OWASP, Keychain, ATS) |
| `xcode-build-resolver` | 빌드 에러 해결 (Swift 6, Privacy Manifest) |
| `xcuitest-runner` | Page Object Model E2E 테스트 |
| `swift-refactor-cleaner` | 리팩토링, 죽은 코드 제거 |
| `ios-doc-updater` | DocC 문서화 |

### 주요 기능

- **자동 SwiftFormat** - Swift 파일 편집 시 자동 포맷팅
- **print() 경고** - 디버그 print문 감지 및 경고
- **세션 지속성** - 세션 간 컨텍스트 유지
- **푸시 전 리뷰** - git push 전 변경사항 확인

---

## 사용 예시

### 에이전트 호출

```
@ios-architect 인증 모듈 아키텍처를 설계해줘
@swift-tdd-guide LoginViewModel 테스트를 작성해줘
@ios-security-reviewer Keychain 구현을 검토해줘
```

### 명령어 사용

```
/plan 사용자 프로필 편집 기능 구현
/swift-review
/xcode-build-fix
```

---

## 지원 아키텍처

### Clean Architecture

```
Presentation Layer (SwiftUI Views, ViewModels)
    ↓
Domain Layer (Use Cases, Entities, Repository Protocols)
    ↓
Data Layer (Repository Implementations, Data Sources)
    ↓
Infrastructure Layer (Network, Database, Keychain)
```

### TCA (The Composable Architecture)

```
Feature
├── State (상태)
├── Action (액션)
├── Reducer (리듀서)
├── Environment/Dependencies (의존성)
└── View (뷰)
```

---

## 테스트 전략

| 테스트 유형 | 프레임워크 | 용도 |
|-------------|------------|------|
| 단위 테스트 | Swift Testing | 비즈니스 로직, ViewModel, Use Case |
| 통합 테스트 | Swift Testing | Repository + Data Source 통합 |
| UI 테스트 | XCUITest | 사용자 흐름, 접근성 |
| 스냅샷 테스트 | swift-snapshot-testing | UI 회귀 |
| 성능 테스트 | XCTest | 메모리, CPU 프로파일링 |
