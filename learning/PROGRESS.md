# iOS Claude Code Learning Progress

> Last Updated: 2026-01-21
> Status: In Progress
> Repository: everything-claude-code-ios

---

## Quick Resume Guide

When starting a new session, tell Claude:
```
Read learning/PROGRESS.md and continue from where we left off.
```

---

## Current Phase: Phase 1.1 - Agents Learning

### Overall Progress

| Phase | Status | Notes File |
|-------|--------|------------|
| Phase 1.1: Agents | 🔄 In Progress | `learning/01-agents-learning.md` |
| Phase 1.2: Rules | ⏳ Pending | `learning/02-rules-learning.md` |
| Phase 1.3: Commands | ⏳ Pending | `learning/03-commands-learning.md` |
| Phase 1.4: Hooks | ⏳ Pending | `learning/04-hooks-learning.md` |
| Phase 1.5: Skills | ⏳ Pending | `learning/05-skills-learning.md` |
| Phase 1.6: MCP Configs | ⏳ Pending | `learning/06-mcp-learning.md` |
| Phase 2: Design | ⏳ Pending | `learning/07-ios-design.md` |
| Phase 3: Implementation | ✅ Started | This repository |

---

## Phase 1.1: Agents Learning Detail

| File | Status | iOS Adaptation | Agent File |
|------|--------|----------------|------------|
| planner.md | ✅ Done | Minor changes | `agents/ios-planner.md` |
| architect.md | ✅ Done | Major changes | `agents/ios-architect.md` |
| tdd-guide.md | ✅ Done | Major changes | `agents/swift-tdd-guide.md` |
| code-reviewer.md | ⏳ Pending | - | `agents/swift-code-reviewer.md` |
| security-reviewer.md | ⏳ Pending | - | `agents/ios-security-reviewer.md` |
| build-error-resolver.md | ⏳ Pending | - | `agents/xcode-build-resolver.md` |
| e2e-runner.md | ⏳ Pending | - | `agents/xcuitest-runner.md` |
| refactor-cleaner.md | ⏳ Pending | - | `agents/swift-refactor-cleaner.md` |
| doc-updater.md | ⏳ Pending | - | `agents/ios-doc-updater.md` |

---

## Key Decisions Made

### 1. Target Platform
- **iOS only** (Swift/SwiftUI)
- Architecture: MVVM + Clean Architecture / TCA

### 2. Output Format
- **Repository**: `everything-claude-code-ios/`
- Learning notes in English

### 3. Focus Areas (All selected)
- Code quality / Review
- Testing (Unit/UI/E2E)
- Build / CI/CD
- Security / App Store Guidelines

### 4. iOS-Specific Additions Confirmed
- iOS Red Flags (Concurrency, SwiftUI, Memory, API/Network, Security)
- App Store Guidelines (2024+ regulations included)
- Clean Architecture layers (with TCA option)
- iOS Testing Strategy (including Performance tests)
- iOS Patterns (Navigation, State Management)
- App Scale Strategy (MVP/Growth/Enterprise)

---

## Repository Structure

```
everything-claude-code-ios/
├── README.md
├── CONTRIBUTING.md
├── LICENSE
├── agents/           # iOS agents (3 complete, 6 skeleton)
├── rules/            # Pending Phase 1.2
├── commands/         # Pending Phase 1.3
├── hooks/            # Pending Phase 1.4
├── skills/           # Pending Phase 1.5
├── mcp-configs/      # Pending Phase 1.6
├── learning/         # Learning notes
└── examples/         # Usage examples
```

---

## Next Steps

1. Continue Phase 1.1: Learn remaining agents (code-reviewer.md, etc.)
2. Apply iOS adaptation analysis
3. Complete skeleton agents with full content
4. Proceed to Phase 1.2 (Rules)

---

## Session Log

| Date | Session | Work Done |
|------|---------|-----------|
| 2026-01-21 | #1 | Project analysis, Plan creation, planner.md & architect.md learning with 2 rounds of senior review |
| 2026-01-21 | #2 | tdd-guide.md learning with Swift Testing, Repository creation with 3 complete + 6 skeleton agents |

---
