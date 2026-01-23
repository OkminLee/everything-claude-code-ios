# Claude Code Performance Optimization

> Optimize Claude Code usage for iOS development workflow

---

## Model Selection Strategy

**Haiku 4.5** (90% of Sonnet capability, 3x cost savings):
- Lightweight agents with frequent invocation
- Pair programming and code generation
- Worker agents in multi-agent systems
- Info.plist / entitlements edits
- Asset catalog additions
- Localization strings (Localizable.strings)
- Simple Extension methods

**Sonnet 4.5** (Best coding model):
- Main development work
- Orchestrating multi-agent workflows
- Complex coding tasks
- SwiftUI View / ViewModel implementation
- Unit / UI test writing
- Build error resolution
- API client implementation

**Opus 4.5** (Deepest reasoning):
- Complex architectural decisions
- Maximum reasoning requirements
- Research and analysis tasks
- TCA / Clean Architecture design
- App modularization strategy
- Complex concurrency issue analysis
- Performance bottleneck analysis (Instruments data)

---

## Context Window Management

Avoid last 20% of context window for:
- Large-scale refactoring
- Feature implementation spanning multiple files
- Debugging complex interactions
- Multi-module Swift project work

Lower context sensitivity tasks:
- Single-file edits
- Independent utility creation
- Documentation updates
- Simple bug fixes
- Extension method additions

### iOS-Specific Context Tips

**High context usage (start fresh or compact):**
- Xcode project restructuring
- Cross-module dependency changes
- SwiftUI navigation refactoring
- Core Data / SwiftData schema changes

**Low context usage (safe to continue):**
- Single View/ViewModel edits
- Adding new model structs
- Writing unit tests for existing code
- Fixing build errors one by one

---

## Ultrathink + Plan Mode

For complex tasks requiring deep reasoning:
1. Use `ultrathink` for enhanced thinking
2. Enable **Plan Mode** for structured approach
3. "Rev the engine" with multiple critique rounds
4. Use split role sub-agents for diverse analysis

### When to Use for iOS Development

| Task | Approach | Why |
|------|----------|-----|
| New feature architecture | Plan Mode + Opus | Multiple valid approaches exist |
| TCA integration | Plan Mode + Ultrathink | Complex state management |
| Performance investigation | Opus | Deep analysis of Instruments data |
| Simple bug fix | Direct (Sonnet) | Clear scope, no planning needed |
| UI polish | Direct (Haiku/Sonnet) | Iterative, fast feedback |

---

## Build Troubleshooting

If build fails:
1. Use **xcode-build-resolver** agent
2. Analyze error messages (Swift compiler, linker, signing)
3. Fix incrementally (one error at a time)
4. Verify after each fix with `xcodebuild`

### Common iOS Build Issues

| Error Type | Agent/Approach | Notes |
|------------|----------------|-------|
| Swift compiler errors | xcode-build-resolver | Type inference, generics |
| Module not found | See dependency guide below | SPM vs CocoaPods differs |
| Signing errors | Manual review needed | Provisioning profiles |
| Linker errors | Check framework linking | Duplicate symbols, missing libs |
| Swift 6 concurrency | xcode-build-resolver | Sendable, actor isolation |

### Dependency Manager Troubleshooting

| Issue | SPM Solution | CocoaPods Solution |
|-------|--------------|-------------------|
| Module not found | Delete `Package.resolved`, rebuild | `pod install --repo-update` |
| Version conflict | Adjust versions in `Package.swift` | Specify version in `Podfile` |
| Binary compatibility | Check `.binaryTarget` platform | Verify `use_frameworks!` |
| Cache issues | `File > Packages > Reset Cache` | `pod cache clean --all` |
| Duplicate symbols | Check for overlapping dependencies | Use `pod deintegrate` and reinstall |

---

## Agent Selection for iOS Tasks

Choose the right agent for efficient Claude Code usage:

| Task | Recommended Agent | Model |
|------|-------------------|-------|
| Architecture design | ios-architect | Opus |
| Code review | swift-code-reviewer | Sonnet |
| TDD workflow | swift-tdd-guide | Sonnet |
| Build errors | xcode-build-resolver | Sonnet |
| E2E tests | xcuitest-runner | Sonnet |
| Dead code cleanup | swift-refactor-cleaner | Sonnet |
| Security audit | ios-security-reviewer | Sonnet/Opus |
| Doc updates | ios-doc-updater | Haiku |

---

## Efficient Workflow Patterns

### 1. Batch Similar Tasks

```
❌ Inefficient: Fix errors one conversation at a time
✅ Efficient: Collect all errors, fix in single session
```

### 2. Use Compact Strategically

```
When to /compact:
- Context > 60% and starting new unrelated task
- After completing a major milestone
- Before complex multi-file changes

When NOT to /compact:
- Mid-debugging session (lose context)
- During iterative UI refinement
- While tracing a specific issue
```

### 3. Parallel Agent Usage

```
For large refactoring:
1. swift-refactor-cleaner → identify dead code
2. swift-code-reviewer → review changes (parallel)
3. swift-tdd-guide → add missing tests (after review)
```

### 4. Context-Aware Requests

```
❌ Vague: "Fix this error"
✅ Specific: "Fix the Swift 6 concurrency error in ProfileViewModel.swift line 45"

❌ Broad: "Review all the code"
✅ Focused: "Review the authentication flow in LoginUseCase.swift for security issues"
```

---

## Project Scale Strategy

Adjust Claude Code usage based on project size:

| Scale | Characteristics | Claude Code Strategy |
|-------|-----------------|---------------------|
| **Small** (< 10 files) | Single module, simple structure | Load full context, work freely |
| **Medium** (10-50 files) | Feature separation begins | Load only working feature, periodic compact |
| **Large** (50+ files) | Multi-module architecture | Work per module, frequent compact, use agents |

### Scale-Specific Tips

**Small Projects:**
- Can reference entire codebase in single session
- No need for frequent compaction
- Direct approach works well

**Medium Projects:**
- Focus on one feature folder at a time
- Compact when switching between features
- Use glob patterns to load relevant files only

**Large Projects:**
- Work within single module boundaries
- Use `ios-architect` for cross-module changes
- Compact after each module completion
- Consider splitting tasks across sessions
- Document module interfaces for context

---

## Quick Reference

### Cost-Effective Model Usage

| Frequency | Task Type | Model |
|-----------|-----------|-------|
| High (100s/day) | Autocomplete, snippets | Haiku |
| Medium (10s/day) | Feature development | Sonnet |
| Low (few/day) | Architecture decisions | Opus |

### Context Usage Guidelines

| Context Level | Recommended Action |
|---------------|-------------------|
| 0-40% | Continue freely |
| 40-60% | Consider scope of next task |
| 60-80% | Compact if switching context |
| 80%+ | Compact before major work |
