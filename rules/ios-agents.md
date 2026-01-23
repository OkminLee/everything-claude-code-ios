# iOS Agent Orchestration

## Available Agents

Located in `~/.claude/agents/`:

| Agent | Purpose | When to Use |
|-------|---------|-------------|
| ios-planner | Implementation planning | Complex features, refactoring |
| ios-architect | System design | Architectural decisions (MVVM, TCA, Clean) |
| swift-tdd-guide | Test-driven development | New features, bug fixes |
| swift-code-reviewer | Code review | After writing code |
| ios-security-reviewer | Security analysis | Before commits |
| xcode-build-resolver | Fix build errors | When Xcode build fails |
| xcuitest-runner | E2E testing | Critical user flows |
| swift-refactor-cleaner | Dead code cleanup | Code maintenance, Swift migration |
| ios-doc-updater | Documentation | Updating DocC docs |

## Immediate Agent Usage

No user prompt needed:
1. Complex feature requests - Use **ios-planner** agent
2. Code just written/modified - Use **swift-code-reviewer** agent
3. Bug fix or new feature - Use **swift-tdd-guide** agent
4. Architectural decision - Use **ios-architect** agent

## When NOT to Use Agents

Avoid over-engineering. Skip agents for:
- **Trivial changes**: Less than 5 lines of code
- **Clear 1:1 fixes**: Typos, off-by-one errors, obvious bugs
- **Following existing patterns**: Code that mirrors existing implementation
- **Simple configuration**: Build settings, plist changes, asset additions
- **Boilerplate code**: Standard CRUD, basic UI setup

**Rule of thumb**: If you can confidently write it in under 2 minutes, just write it.

## Parallel Task Execution

ALWAYS use parallel Task execution for independent operations:

```markdown
# GOOD: Parallel execution
Launch 3 agents in parallel:
1. Agent 1: Security analysis of AuthManager.swift
2. Agent 2: Performance review of CacheService.swift
3. Agent 3: Concurrency check of NetworkClient.swift

# BAD: Sequential when unnecessary
First agent 1, then agent 2, then agent 3
```

## Multi-Perspective Analysis

For complex iOS problems, use split role sub-agents:

### General Perspectives
- Factual reviewer
- Senior iOS engineer
- Consistency reviewer
- Redundancy checker

### iOS-Specific Perspectives
- **Concurrency expert**: Swift 6 strict concurrency, @Sendable, actor isolation, MainActor
- **Memory management expert**: Retain cycles, weak self, closure captures, Instruments leaks
- **App Store compliance checker**: Review Guidelines, Privacy Manifest, Entitlements
- **Platform compatibility reviewer**: iOS version availability, device capabilities, deprecations
- **Accessibility specialist**: VoiceOver, Dynamic Type, color contrast
- **Performance profiler**: Instruments analysis (Time Profiler, Allocations, Leaks, Energy Log)
- **Localization reviewer**: String extraction, RTL layout, pluralization, date/number formatting
- **Extension specialist**: Widget memory limits (30MB), App Groups, shared containers, extension lifecycle

## Agent Chaining Patterns

### Feature Development Chain
```
ios-planner → ios-architect → swift-tdd-guide → swift-code-reviewer
```

### Bug Fix Chain
```
(investigate/reproduce) → swift-tdd-guide → swift-code-reviewer
                              └─ xcode-build-resolver (if build error)
```
Note: Start with investigation. Use xcode-build-resolver only for compilation errors.

### Release Preparation Chain
```
swift-refactor-cleaner → ios-security-reviewer → ios-doc-updater
```

### Code Quality Chain (Parallel)
```
┌─ swift-code-reviewer
├─ ios-security-reviewer
└─ swift-refactor-cleaner (dead code check)
```

## Context-Aware Agent Selection

| Situation | Primary Agent | Supporting Agents |
|-----------|---------------|-------------------|
| New screen/feature | ios-planner | ios-architect, swift-tdd-guide |
| Crash/bug report | xcode-build-resolver | swift-code-reviewer |
| PR review request | swift-code-reviewer | ios-security-reviewer |
| Performance issue | swift-code-reviewer | swift-refactor-cleaner |
| App Store rejection | ios-security-reviewer | ios-doc-updater |
| iOS version upgrade | swift-refactor-cleaner | xcode-build-resolver |
| SwiftUI migration | ios-architect | swift-refactor-cleaner |
