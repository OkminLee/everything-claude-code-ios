# iOS Claude Code Plugin

> Claude Code plugin optimized for iOS/Swift development.

[![Swift](https://img.shields.io/badge/Swift-5.9+-orange.svg)](https://swift.org)
[![Platform](https://img.shields.io/badge/Platform-iOS%2017+-blue.svg)](https://developer.apple.com/ios/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Plugin-blueviolet.svg)](https://claude.ai/code)

> **Based on [everything-claude-code](https://github.com/affaan-m/everything-claude-code)** by [@affaan-m](https://github.com/affaan-m)
> An Anthropic hackathon winner's comprehensive Claude Code configuration collection.

---

## Quick Start

### Prerequisites

- [Claude Code CLI](https://claude.ai/code) installed and configured
- macOS (required for iOS development)
- Git
- Xcode 15+ (for iOS project development)

### Installation

Choose the method that best fits your workflow:

#### Method A: Global Installation

Install once, use across all your iOS projects.

```bash
# Clone the repository
git clone https://github.com/mininny/everything-claude-code-ios.git

# Copy to Claude Code plugins directory
cp -r everything-claude-code-ios ~/.claude/plugins/everything-claude-code-ios
```

**When to use**: You want consistent iOS development assistance across all projects.

#### Method B: Project-Local Installation

Install directly in your project for project-specific customization.

```bash
# Navigate to your iOS project root
cd /path/to/your/ios-project

# Clone into the .claude/plugins directory
git clone https://github.com/mininny/everything-claude-code-ios.git .claude/plugins/ios
```

**When to use**: You want to customize the plugin per project or share settings with your team via version control.

### Verify Installation

After installation, start Claude Code in your project directory:

```bash
claude
```

Then try a command to verify the plugin is loaded:

```
/plan Test the plugin installation
```

If you see the planning agent respond, the installation was successful.

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

## Troubleshooting

### Plugin Not Recognized

**Symptoms**: Commands like `/plan` don't work, agents aren't available.

**Solutions**:
1. Verify the plugin directory exists:
   ```bash
   # For global installation
   ls ~/.claude/plugins/everything-claude-code-ios

   # For local installation
   ls .claude/plugins/ios
   ```
2. Check file permissions:
   ```bash
   chmod -R 755 ~/.claude/plugins/everything-claude-code-ios
   ```
3. Restart Claude Code completely (exit and relaunch)

### MCP Servers Not Connecting

**Symptoms**: GitHub integration not working, memory features unavailable.

**Solutions**:
1. Verify environment variables are set:
   ```bash
   echo $GITHUB_TOKEN
   ```
2. Check if the token has required permissions (repo, read:user)
3. Regenerate the token if expired

### Hooks Not Triggering

**Symptoms**: SwiftFormat doesn't run on save, pre-push review doesn't appear.

**Solutions**:
1. Verify `hooks.json` syntax is valid:
   ```bash
   cat ~/.claude/plugins/everything-claude-code-ios/hooks.json | python3 -m json.tool
   ```
2. Check if the file pattern matches your files
3. Ensure the hook command (e.g., `swiftformat`) is installed and in PATH

### Swift Tools Not Found

**Symptoms**: SwiftFormat, Periphery, or other tools report "command not found".

**Solutions**:
1. Install missing tools via Homebrew:
   ```bash
   brew install swiftformat
   brew install peripheryapp/periphery/periphery
   ```
2. Verify installation:
   ```bash
   which swiftformat
   which periphery
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

### Origin Project

This iOS-focused plugin is built upon **[everything-claude-code](https://github.com/affaan-m/everything-claude-code)**, created by [@affaan-m](https://github.com/affaan-m).

The original project provides production-ready agents, skills, hooks, commands, and MCP configurations developed through extensive real-world usage. It won recognition at an Anthropic hackathon and serves as a comprehensive foundation for Claude Code customization.

We deeply appreciate the open-source contribution that made this iOS adaptation possible.

### Additional Thanks

- Apple and the Swift community for iOS development best practices
- The Claude Code team for building an extensible platform

---

<div align="center">

# 한국어 가이드

</div>

---

## 빠른 시작

### 필수 조건

- [Claude Code CLI](https://claude.ai/code) 설치 및 설정 완료
- macOS (iOS 개발에 필요)
- Git
- Xcode 15+ (iOS 프로젝트 개발용)

### 설치 방법

사용 환경에 맞는 방법을 선택하세요:

#### 방법 A: 글로벌 설치

한 번 설치하면 모든 iOS 프로젝트에서 사용 가능합니다.

```bash
# 저장소 클론
git clone https://github.com/mininny/everything-claude-code-ios.git

# Claude Code 플러그인 디렉토리에 복사
cp -r everything-claude-code-ios ~/.claude/plugins/everything-claude-code-ios
```

**사용 시기**: 모든 프로젝트에서 일관된 iOS 개발 지원을 원할 때

#### 방법 B: 프로젝트별 설치

프로젝트에 직접 설치하여 프로젝트별 커스터마이징이 가능합니다.

```bash
# iOS 프로젝트 루트로 이동
cd /path/to/your/ios-project

# .claude/plugins 디렉토리에 클론
git clone https://github.com/mininny/everything-claude-code-ios.git .claude/plugins/ios
```

**사용 시기**: 프로젝트별로 플러그인을 커스터마이징하거나, 팀원과 버전 관리를 통해 설정을 공유하고 싶을 때

### 설치 확인

설치 후, 프로젝트 디렉토리에서 Claude Code를 시작합니다:

```bash
claude
```

플러그인이 로드되었는지 확인하려면 명령어를 실행해보세요:

```
/plan 플러그인 설치 테스트
```

플래닝 에이전트가 응답하면 설치가 완료된 것입니다.

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

## 문제 해결

### 플러그인이 인식되지 않음

**증상**: `/plan` 같은 명령어가 작동하지 않거나 에이전트를 사용할 수 없음

**해결 방법**:
1. 플러그인 디렉토리 존재 확인:
   ```bash
   # 글로벌 설치의 경우
   ls ~/.claude/plugins/everything-claude-code-ios

   # 로컬 설치의 경우
   ls .claude/plugins/ios
   ```
2. 파일 권한 확인:
   ```bash
   chmod -R 755 ~/.claude/plugins/everything-claude-code-ios
   ```
3. Claude Code 완전히 재시작 (종료 후 다시 실행)

### MCP 서버 연결 실패

**증상**: GitHub 연동이 안 되거나 메모리 기능을 사용할 수 없음

**해결 방법**:
1. 환경 변수 설정 확인:
   ```bash
   echo $GITHUB_TOKEN
   ```
2. 토큰에 필요한 권한이 있는지 확인 (repo, read:user)
3. 토큰이 만료되었다면 재발급

### Hooks가 동작하지 않음

**증상**: SwiftFormat이 저장 시 실행되지 않거나, 푸시 전 리뷰가 나타나지 않음

**해결 방법**:
1. `hooks.json` 문법 확인:
   ```bash
   cat ~/.claude/plugins/everything-claude-code-ios/hooks.json | python3 -m json.tool
   ```
2. 파일 패턴이 올바른지 확인
3. Hook 명령어(예: `swiftformat`)가 설치되어 있고 PATH에 있는지 확인

### Swift 도구를 찾을 수 없음

**증상**: SwiftFormat, Periphery 등의 도구가 "command not found" 오류 발생

**해결 방법**:
1. Homebrew로 누락된 도구 설치:
   ```bash
   brew install swiftformat
   brew install peripheryapp/periphery/periphery
   ```
2. 설치 확인:
   ```bash
   which swiftformat
   which periphery
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

---

## 설정

### 환경 변수

MCP 서버 사용을 위해 다음 환경 변수를 설정하세요:

```bash
export GITHUB_TOKEN="your_github_pat"
```

### 커스터마이징

프로젝트의 `.claude/settings.json`에서 플러그인 설정을 재정의할 수 있습니다:

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

## 기여하기

기여를 환영합니다! 가이드라인은 [CONTRIBUTING.md](CONTRIBUTING.md)를 참고하세요.

---

## 라이선스

이 프로젝트는 MIT 라이선스를 따릅니다. 자세한 내용은 [LICENSE](LICENSE) 파일을 참고하세요.

---

## 감사의 말

### 원본 프로젝트

이 iOS 전용 플러그인은 [@affaan-m](https://github.com/affaan-m)이 만든 **[everything-claude-code](https://github.com/affaan-m/everything-claude-code)**를 기반으로 합니다.

원본 프로젝트는 Anthropic 해커톤에서 수상한 작품으로, 실제 사용 경험을 바탕으로 개발된 프로덕션 수준의 에이전트, 스킬, 훅, 명령어, MCP 설정을 제공합니다. Claude Code 커스터마이징의 종합적인 기반이 되는 프로젝트입니다.

이 iOS 버전을 만들 수 있게 해준 오픈소스 기여에 깊이 감사드립니다.

### 추가 감사

- iOS 개발 모범 사례를 제공하는 Apple과 Swift 커뮤니티
- 확장 가능한 플랫폼을 구축한 Claude Code 팀
