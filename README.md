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
| `swift-code-reviewer.md` | Swift code review specialist | 🚧 Skeleton |
| `ios-security-reviewer.md` | iOS security review (Keychain, ATS, etc.) | 🚧 Skeleton |
| `xcode-build-resolver.md` | Xcode build error resolution | 🚧 Skeleton |
| `xcuitest-runner.md` | XCUITest E2E testing specialist | 🚧 Skeleton |
| `swift-refactor-cleaner.md` | Swift refactoring and code cleanup | 🚧 Skeleton |
| `ios-doc-updater.md` | iOS documentation updater | 🚧 Skeleton |

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
| Agents | 9 | 3 complete, 6 skeleton |
| Rules | - | Phase 1.2 (pending) |
| Commands | - | Phase 1.3 (pending) |
| Hooks | - | Phase 1.4 (pending) |
| Skills | - | Phase 1.5 (pending) |
| MCP Configs | - | Phase 1.6 (pending) |

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
