# iOS Research Context

Mode: Exploration, investigation, learning
Focus: Understanding iOS APIs and patterns before coding

## Behavior

- Read Apple documentation first
- Check iOS version availability (@available)
- Look for WWDC sessions on the topic
- Document findings with code examples
- Don't write code until understanding is clear

## Research Process

1. Understand the requirement
2. Check Apple Developer Documentation
3. Explore existing codebase patterns
4. Find sample code / WWDC sessions
5. Verify API availability (iOS version)
6. Summarize with implementation notes

## Tools to Favor

- Read for understanding Swift code
- Grep, Glob for finding patterns in codebase
- WebSearch for Apple docs, Swift forums
- WebFetch for developer.apple.com
- Context7 for third-party library documentation

## Key Resources

### Official Apple

- developer.apple.com/documentation
- developer.apple.com/videos (WWDC)
- Swift.org documentation
- Swift Evolution proposals (github.com/apple/swift-evolution)

### Community

- hackingwithswift.com (practical tutorials)
- swiftbysundell.com (in-depth articles)
- avanderlee.com (SwiftUI/Combine deep dives)
- pointfreeco.com (advanced Swift patterns)
- forums.swift.org (Swift language discussions)

### Tools

- Dash app (offline documentation)
- SF Symbols app (icon reference)
- Accessibility Inspector (a11y testing)

## Search Tips

```
# Apple documentation search
site:developer.apple.com [API name]

# WWDC session search
"WWDC24 [topic]"
"WWDC23 [topic]"

# Swift forums
site:forums.swift.org [question]

# Stack Overflow iOS
[ios] [swift] [topic] site:stackoverflow.com
```

## API Availability Check

```swift
// Check before using new APIs
if #available(iOS 17, *) {
    // Use new API
} else {
    // Fallback for older iOS
}

// Mark entire function
@available(iOS 16, *)
func newFeature() { }
```

## Output

Findings first, recommendations second.
Include code snippets when relevant.
Note iOS version requirements clearly.
