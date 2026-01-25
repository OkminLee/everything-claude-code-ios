# iOS Development Context

Mode: Active SwiftUI/UIKit development
Focus: Implementation, building features

## Behavior

- Write Swift code first, explain after
- Prefer working solutions over perfect solutions
- Build after changes (xcodebuild or Xcode)
- Keep commits atomic
- Follow Apple Human Interface Guidelines (HIG)

## Priorities

1. Get it building (no Xcode errors/warnings)
2. Get it working (functionality verified)
3. Get it right (edge cases, error handling)
4. Get it clean (refactor, optimize)

## Tools to Favor

- Edit, Write for Swift code changes
- Bash for xcodebuild, swift test, swift build
- Grep, Glob for finding Swift patterns
- LSP for symbol navigation and refactoring

## Swift-Specific

- Use Swift Concurrency (async/await) over completion handlers
- Prefer @MainActor for UI updates over DispatchQueue.main
- Use structured concurrency with TaskGroup when appropriate
- Leverage Swift's type system (enums, protocols, generics)
- Use guard for early exits, avoid deep nesting

## SwiftUI vs UIKit

- New features: SwiftUI first (iOS 15+)
- Complex animations: Consider UIKit
- Legacy integration: UIViewRepresentable/UIViewControllerRepresentable
- Navigation: NavigationStack (iOS 16+) or NavigationView

## Dependency Management

- Prefer Swift Package Manager (SPM) over CocoaPods
- Lock package versions in Package.resolved
- Avoid version conflicts across packages
- Minimize external dependencies when possible

## Build Commands

```bash
# Build for simulator
xcodebuild -scheme MyApp -destination 'platform=iOS Simulator,name=iPhone 15'

# Run tests
xcodebuild test -scheme MyApp -destination 'platform=iOS Simulator,name=iPhone 15'

# Swift package commands
swift build
swift test
```

## iOS-Specific Considerations

- Check iOS version compatibility (@available)
- Consider device variations (iPhone/iPad/orientation)
- Handle app lifecycle properly (scenePhase, background tasks)
- Test on Simulator before real device
- Use SwiftUI Previews for rapid iteration
