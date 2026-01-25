# iOS Claude Code Configuration

> Claude Code CLI configuration optimized for iOS/Swift development.

[![Swift](https://img.shields.io/badge/Swift-5.9+-orange.svg)](https://swift.org)
[![Platform](https://img.shields.io/badge/Platform-iOS%2017+-blue.svg)](https://developer.apple.com/ios/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

---

## Overview

This repository provides a comprehensive set of Claude Code configurations specifically designed for iOS development. It includes agents, rules, commands, and hooks tailored for Swift/SwiftUI projects following modern iOS development practices.

### Key Features

- **iOS-Specific Agents** - Planning, architecture, TDD, code review optimized for Swift
- **Swift Testing Support** - Modern testing framework with `@Test`, `@Suite`, `#expect`
- **Clean Architecture / TCA** - Support for both architectural patterns
- **App Store Guidelines** - Compliance checks including 2024+ regulations
- **XCUITest E2E** - End-to-end testing with Page Object Model

---

## Agents

| Agent | Description | Status |
|-------|-------------|--------|
| `ios-planner.md` | iOS feature planning with Red Flags and App Store checks | ✅ Complete |
| `ios-architect.md` | iOS architecture design with Clean Architecture/TCA patterns | ✅ Complete |
| `swift-tdd-guide.md` | Swift Testing TDD guide with Actor-based mocking | ✅ Complete |
| `swift-code-reviewer.md` | Swift code review specialist (328 lines) | ✅ Complete |
| `ios-security-reviewer.md` | iOS security review - OWASP Top 10, Keychain, ATS (648 lines) | ✅ Complete |
| `xcode-build-resolver.md` | Xcode build error resolution - Swift 6, Privacy Manifest (1077 lines) | ✅ Complete |
| `xcuitest-runner.md` | XCUITest E2E testing - Screen Object Pattern, CI/CD (1437 lines) | ✅ Complete |
| `swift-refactor-cleaner.md` | Swift refactoring - Periphery, dead code cleanup (528 lines) | ✅ Complete |
| `ios-doc-updater.md` | iOS documentation - DocC, Codemaps (808 lines) | ✅ Complete |

---

## Quick Start

### 1. Clone the repository

```bash
git clone https://github.com/yourusername/everything-claude-code-ios.git
```

### 2. Copy agents to your project

```bash
cp -r everything-claude-code-ios/agents /path/to/your/ios-project/.claude/agents
```

### 3. Use in Claude Code

```
@ios-planner Plan the implementation of a new feature
@ios-architect Design the architecture for this module
@swift-tdd-guide Guide me through TDD for this feature
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

## Learning Notes

The `learning/` directory contains detailed notes from studying the original [everything-claude-code](https://github.com/anthropics/everything-claude-code) repository:

- `PROGRESS.md` - Overall learning progress tracker
- `01-agents-learning.md` - Detailed agent analysis with iOS adaptations

---

## Project Status

| Category | Files | Status |
|----------|-------|--------|
| Agents | 9 | ✅ 9/9 Complete |
| Rules | 8 | ✅ Complete |
| Commands | 10 | ✅ Complete |
| Hooks | 1 + scripts | ✅ Complete |
| Skills | 8+ | ✅ Complete |
| MCP Configs | 1 | ✅ Complete |
| Contexts | 3 | ✅ Complete |
| Examples | - | 📝 In Progress |

### Completed Components

**Rules** (8 files):
- `swift-style.md` - Swift API Design Guidelines, Concurrency
- `swiftui-patterns.md` - SwiftUI best practices, iOS 17+ patterns
- `ios-testing.md` - Swift Testing, XCTest, coverage targets
- `ios-security.md` - OWASP, Keychain, Privacy Manifest
- `ios-performance.md` - Instruments, optimization
- `ios-git-workflow.md` - Commit, PR conventions
- `ios-hooks.md` - Hook usage guide
- `ios-agents.md` - Agent orchestration

**Skills** (8+ files):
- `swift-tdd-workflow/` - TDD workflow with Swift Testing
- `ios-security-review/` - Security review skill
- `continuous-learning/` - Session pattern extraction
- `strategic-compact/` - Manual compaction suggestions
- `ios-project-guidelines.md`, `swiftui-patterns.md`, `swift-coding-standards.md`, `backend-patterns.md`

**Hooks** (`ios-hooks.json`):
- Pre-push review pause
- SwiftFormat auto-formatting
- print() statement warnings
- Memory persistence (session start/end)
- Strategic compact suggestions

**MCP Configs** (`ios-mcp-servers.json`):
- GitHub, Memory, Sequential-thinking, Context7, Filesystem

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
