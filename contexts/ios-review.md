# iOS Code Review Context

Mode: PR review, Swift code analysis
Focus: Quality, security, performance, Apple guidelines

## Behavior

- Read thoroughly before commenting
- Prioritize by severity (critical > high > medium > low)
- Suggest fixes with Swift code examples
- Check for iOS-specific issues
- Consider backward compatibility

## Review Checklist

### Critical

- [ ] Retain cycles / memory leaks (weak self, unowned)
- [ ] Main thread UI violations (@MainActor missing)
- [ ] Force unwrapping without safety (!, try!)
- [ ] Hardcoded secrets or API keys
- [ ] Task/Combine subscriptions not cancelled
- [ ] Background task completion not handled
- [ ] Infinite loops / deadlock potential
- [ ] Data race conditions (actor isolation)

### High

- [ ] Missing error handling (do-catch, Result)
- [ ] iOS version compatibility (@available)
- [ ] Accessibility missing (VoiceOver, Dynamic Type)
- [ ] App lifecycle not handled (scenePhase)
- [ ] @StateObject vs @ObservedObject misuse
- [ ] Large image memory not released (autoreleasepool)
- [ ] Network calls without timeout/retry
- [ ] Keychain usage for sensitive data

### Medium

- [ ] Performance (lazy loading, caching)
- [ ] Code organization (MVVM, Clean Architecture)
- [ ] Swift naming conventions (API Design Guidelines)
- [ ] Unit test coverage
- [ ] DispatchQueue.main.async overuse
- [ ] Combine publishers not using receive(on:)
- [ ] Missing loading/error states in UI

### Low

- [ ] Code style consistency (SwiftLint rules)
- [ ] Documentation comments (///)
- [ ] TODO/FIXME cleanup
- [ ] Unused imports/variables
- [ ] Magic numbers without constants

## Common Issues & Fixes

### Retain Cycle

```swift
// Bad
Timer.scheduledTimer(withTimeInterval: 1, repeats: true) { _ in
    self.update() // Strong reference
}

// Good
Timer.scheduledTimer(withTimeInterval: 1, repeats: true) { [weak self] _ in
    self?.update()
}
```

### MainActor Violation

```swift
// Bad
func fetchData() async {
    let data = await api.fetch()
    self.items = data // UI update off main thread
}

// Good
@MainActor
func fetchData() async {
    let data = await api.fetch()
    self.items = data
}
```

### StateObject vs ObservedObject

```swift
// Bad - recreates VM on every parent update
struct MyView: View {
    @ObservedObject var viewModel = ViewModel()
}

// Good - owns the VM lifecycle
struct MyView: View {
    @StateObject var viewModel = ViewModel()
}

// Good - receives VM from parent
struct ChildView: View {
    @ObservedObject var viewModel: ViewModel
}
```

### Task Cancellation

```swift
// Bad - task continues after view disappears
.task {
    await loadData()
}

// Good - explicit cancellation check
.task {
    guard !Task.isCancelled else { return }
    await loadData()
}
```

## Output Format

Group findings by file, severity first.
Include Swift code suggestions for fixes.
Reference Apple documentation when relevant.

```markdown
## file.swift

### Critical
- Line 42: Retain cycle in closure
  - Fix: Add `[weak self]` capture

### High
- Line 78: Missing @MainActor
  - Fix: Add @MainActor to function declaration
```
