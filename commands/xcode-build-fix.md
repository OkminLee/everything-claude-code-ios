---
description: Incrementally fix Xcode build and Swift compiler errors. Parse errors, explain issues, apply fixes, verify resolution.
---

# Xcode Build Fix

Incrementally fix Swift compiler and Xcode build errors.

This command invokes the **xcode-build-resolver** agent.

## Process

1. Run build:
   ```bash
   xcodebuild build \
     -scheme MyApp \
     -destination 'platform=iOS Simulator,name=iPhone 16' \
     2>&1 | xcpretty
   ```

   Or for SPM:
   ```bash
   swift build 2>&1
   ```

2. Parse error output:
   - Group by file
   - Sort by severity (error > warning)
   - Identify error patterns

3. For each error:
   - Show error context (5 lines before/after)
   - Explain the issue
   - Propose fix
   - Apply fix
   - Re-run build
   - Verify error resolved

4. Stop if:
   - Fix introduces new errors
   - Same error persists after 3 attempts
   - User requests pause
   - Circular dependency detected

5. Show summary:
   - Errors fixed
   - Errors remaining
   - New errors introduced
   - Warnings to address

## Common Error Patterns

### Swift 6 Concurrency

```
error: sending 'self.data' risks causing data races
```
**Fix**: Add `@MainActor`, use `actor`, or make type `Sendable`

```
error: call to main actor-isolated method cannot be made from nonisolated context
```
**Fix**: Add `await MainActor.run { }` or mark caller `@MainActor`

### Type Errors

```
error: cannot convert value of type 'X' to expected argument type 'Y'
```
**Fix**: Check type compatibility, add conversion, or fix generic constraints

```
error: value of optional type 'X?' must be unwrapped
```
**Fix**: Add `guard let`, `if let`, or nil-coalescing `??`

### Module/Import Errors

```
error: no such module 'ModuleName'
```
**Fix**: Check SPM dependencies, pod install, or Framework search paths

```
error: cannot find 'TypeName' in scope
```
**Fix**: Add import statement or check access level

### Xcode Project Issues

```
error: multiple commands produce 'X'
```
**Fix**: Check duplicate file references in Build Phases

```
error: provisioning profile doesn't match
```
**Fix**: Update signing settings in Signing & Capabilities

## Fix Strategy by Category

| Category | Approach |
|----------|----------|
| Type mismatch | Check expected vs actual types, add conversions |
| Optional handling | Use guard/if let, avoid force unwrap |
| Concurrency | Add isolation annotations, use actors |
| Missing import | Add import, check dependency graph |
| Access control | Adjust public/internal/private |
| Protocol conformance | Implement required methods |
| Generic constraints | Add where clauses or type constraints |

## Running Builds

### Xcode Project

```bash
# Build only (fast feedback)
xcodebuild build \
  -scheme MyApp \
  -destination 'platform=iOS Simulator,name=iPhone 16'

# Build with clean
xcodebuild clean build \
  -scheme MyApp \
  -destination 'platform=iOS Simulator,name=iPhone 16'

# Analyze for warnings
xcodebuild analyze \
  -scheme MyApp \
  -destination 'platform=iOS Simulator,name=iPhone 16'
```

### Swift Package

```bash
# Build
swift build

# Build with verbose output
swift build -v

# Build for release
swift build -c release
```

## Error Output Format

When reporting fixed errors:

```
## Build Fix Report

### ✅ Fixed (3)

1. **Type Mismatch** - `HomeViewModel.swift:45`
   - Error: Cannot convert '[Item]' to 'Set<Item>'
   - Fix: Changed `items` type from `Set` to `Array`

2. **Missing Actor Isolation** - `DataService.swift:23`
   - Error: Call to main actor-isolated method
   - Fix: Added `@MainActor` to `updateUI()` method

3. **Optional Unwrap** - `UserManager.swift:89`
   - Error: Value of optional type must be unwrapped
   - Fix: Added `guard let` with early return

### ⚠️ Warnings (1)

1. **Deprecated API** - `NetworkClient.swift:56`
   - Warning: 'URLSession.shared.dataTask' is deprecated
   - Suggestion: Migrate to async/await version

### ❌ Remaining (0)

Build succeeded!
```

## Safety Rules

- Fix **one error at a time**
- Re-build after each fix
- Don't change unrelated code
- Preserve existing functionality
- Keep original error message in comments if complex fix

## Integration with Other Commands

- Use `/swift-review` after fixes to check code quality
- Use `/swift-tdd` if fixes need test coverage
- Use `/ios-coverage` to verify no regression

## Related Agents

This command invokes the `xcode-build-resolver` agent located at:
`~/.claude/agents/xcode-build-resolver.md`
