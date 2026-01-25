---
description: Enforce test-driven development workflow for Swift/iOS. Define protocols, write tests FIRST using Swift Testing, then implement minimal code. Ensure 80%+ coverage.
---

# Swift TDD Command

This command invokes the **swift-tdd-guide** agent to enforce test-driven development methodology for iOS projects.

## What This Command Does

1. **Define Protocols** - Establish contracts and interfaces first
2. **Write Tests First** - Create failing tests using Swift Testing (RED)
3. **Implement Minimal Code** - Write just enough to pass (GREEN)
4. **Refactor** - Improve code while keeping tests green (REFACTOR)
5. **Verify Coverage** - Ensure 80%+ test coverage

## When to Use

Use `/swift-tdd` when:
- Implementing new iOS features
- Adding new ViewModels, UseCases, or Repositories
- Fixing bugs (write test that reproduces bug first)
- Refactoring existing Swift code
- Building critical business logic

## How It Works

The swift-tdd-guide agent will:

1. **Define protocols** for dependencies and contracts
2. **Write tests that will FAIL** (because code doesn't exist yet)
3. **Run tests** and verify they fail for the right reason
4. **Write minimal implementation** to make tests pass
5. **Run tests** and verify they pass
6. **Refactor** code while keeping tests green
7. **Check coverage** and add more tests if below 80%

## TDD Cycle

```
RED → GREEN → REFACTOR → REPEAT

RED:      Write a failing test using #expect
GREEN:    Write minimal code to pass
REFACTOR: Improve code, keep tests passing
REPEAT:   Next feature/scenario
```

## Example Usage

```
User: /swift-tdd I need a use case to calculate market liquidity score

Agent (swift-tdd-guide):
# TDD Session: Market Liquidity Score Calculator

## Step 1: Define Protocol (SCAFFOLD)

```swift
// Domain/UseCases/LiquidityCalculator.swift
import Foundation

struct MarketData: Sendable {
    let totalVolume: Double
    let bidAskSpread: Double
    let activeTraders: Int
    let lastTradeTime: Date
}

protocol LiquidityCalculating: Sendable {
    func calculateScore(for market: MarketData) -> Double
}

// Placeholder implementation
struct LiquidityCalculator: LiquidityCalculating {
    func calculateScore(for market: MarketData) -> Double {
        fatalError("Not implemented")
    }
}
```

## Step 2: Write Failing Test (RED)

```swift
// Tests/Domain/LiquidityCalculatorTests.swift
import Testing
@testable import MyApp

@Suite struct LiquidityCalculatorTests {
    let sut = LiquidityCalculator()

    @Test func calculateScore_forLiquidMarket_returnsHighScore() {
        // Given
        let market = MarketData(
            totalVolume: 100_000,
            bidAskSpread: 0.01,
            activeTraders: 500,
            lastTradeTime: Date()
        )

        // When
        let score = sut.calculateScore(for: market)

        // Then
        #expect(score > 80)
        #expect(score <= 100)
    }

    @Test func calculateScore_forIlliquidMarket_returnsLowScore() {
        // Given
        let market = MarketData(
            totalVolume: 100,
            bidAskSpread: 0.5,
            activeTraders: 2,
            lastTradeTime: Date().addingTimeInterval(-86400) // 1 day ago
        )

        // When
        let score = sut.calculateScore(for: market)

        // Then
        #expect(score < 30)
        #expect(score >= 0)
    }

    @Test func calculateScore_withZeroVolume_returnsZero() {
        // Given
        let market = MarketData(
            totalVolume: 0,
            bidAskSpread: 0,
            activeTraders: 0,
            lastTradeTime: Date()
        )

        // When
        let score = sut.calculateScore(for: market)

        // Then
        #expect(score == 0)
    }
}
```

## Step 3: Run Tests - Verify FAIL

```bash
xcodebuild test \
  -scheme MyApp \
  -destination 'platform=iOS Simulator,name=iPhone 16'

Test Suite 'LiquidityCalculatorTests' started
  ✕ calculateScore_forLiquidMarket_returnsHighScore
    Fatal error: Not implemented

1 test failed, 0 passed
```

✅ Tests fail as expected. Ready to implement.

## Step 4: Implement Minimal Code (GREEN)

```swift
// Domain/UseCases/LiquidityCalculator.swift
struct LiquidityCalculator: LiquidityCalculating {
    func calculateScore(for market: MarketData) -> Double {
        // Handle zero volume edge case
        guard market.totalVolume > 0 else { return 0 }

        // Calculate component scores (0-100 scale)
        let volumeScore = min(market.totalVolume / 1000, 100)
        let spreadScore = max(100 - (market.bidAskSpread * 1000), 0)
        let traderScore = min(Double(market.activeTraders) / 10, 100)

        // Recent activity bonus
        let hoursSinceLastTrade = Date().timeIntervalSince(market.lastTradeTime) / 3600
        let recencyScore = max(100 - (hoursSinceLastTrade * 10), 0)

        // Weighted average
        let score = volumeScore * 0.4 +
                    spreadScore * 0.3 +
                    traderScore * 0.2 +
                    recencyScore * 0.1

        return min(max(score, 0), 100)
    }
}
```

## Step 5: Run Tests - Verify PASS

```bash
xcodebuild test \
  -scheme MyApp \
  -destination 'platform=iOS Simulator,name=iPhone 16'

Test Suite 'LiquidityCalculatorTests' passed
  ✓ calculateScore_forLiquidMarket_returnsHighScore (0.003 seconds)
  ✓ calculateScore_forIlliquidMarket_returnsLowScore (0.002 seconds)
  ✓ calculateScore_withZeroVolume_returnsZero (0.001 seconds)

3 tests passed
```

✅ All tests passing!

## Step 6: Refactor (IMPROVE)

```swift
// Domain/UseCases/LiquidityCalculator.swift
struct LiquidityCalculator: LiquidityCalculating {
    private enum Weight {
        static let volume: Double = 0.4
        static let spread: Double = 0.3
        static let traders: Double = 0.2
        static let recency: Double = 0.1
    }

    private enum ScaleFactor {
        static let volume: Double = 1000
        static let spread: Double = 1000
        static let traders: Double = 10
        static let recencyPenalty: Double = 10
    }

    func calculateScore(for market: MarketData) -> Double {
        guard market.totalVolume > 0 else { return 0 }

        let volumeScore = componentScore(market.totalVolume / ScaleFactor.volume)
        let spreadScore = componentScore(100 - market.bidAskSpread * ScaleFactor.spread)
        let traderScore = componentScore(Double(market.activeTraders) / ScaleFactor.traders)
        let recencyScore = calculateRecencyScore(for: market.lastTradeTime)

        let weightedScore = volumeScore * Weight.volume +
                            spreadScore * Weight.spread +
                            traderScore * Weight.traders +
                            recencyScore * Weight.recency

        return clamp(weightedScore)
    }

    private func calculateRecencyScore(for lastTradeTime: Date) -> Double {
        let hoursSinceLastTrade = Date().timeIntervalSince(lastTradeTime) / 3600
        return componentScore(100 - hoursSinceLastTrade * ScaleFactor.recencyPenalty)
    }

    private func componentScore(_ value: Double) -> Double {
        clamp(value)
    }

    private func clamp(_ value: Double) -> Double {
        min(max(value, 0), 100)
    }
}
```

## Step 7: Verify Tests Still Pass

```bash
xcodebuild test -scheme MyApp -destination 'platform=iOS Simulator,name=iPhone 16'

Test Suite 'LiquidityCalculatorTests' passed
  ✓ calculateScore_forLiquidMarket_returnsHighScore
  ✓ calculateScore_forIlliquidMarket_returnsLowScore
  ✓ calculateScore_withZeroVolume_returnsZero

3 tests passed
```

✅ Refactoring complete, tests still passing!

## Step 8: Check Coverage

```bash
xcodebuild test \
  -scheme MyApp \
  -destination 'platform=iOS Simulator,name=iPhone 16' \
  -enableCodeCoverage YES

xcrun xccov view --report Build/Logs/Test/*.xcresult

LiquidityCalculator.swift    100.0%  (12/12 lines)
```

✅ TDD session complete!
```

## TDD Best Practices

**DO:**
- ✅ Write the test FIRST, before any implementation
- ✅ Run tests and verify they FAIL before implementing
- ✅ Write minimal code to make tests pass
- ✅ Refactor only after tests are green
- ✅ Add edge cases and error scenarios
- ✅ Aim for 80%+ coverage (100% for critical code)
- ✅ Use Swift Testing for new projects (iOS 17+)

**DON'T:**
- ❌ Write implementation before tests
- ❌ Skip running tests after each change
- ❌ Write too much code at once
- ❌ Ignore failing tests
- ❌ Test implementation details (test behavior)
- ❌ Mock everything (prefer integration tests)
- ❌ Mix Swift Testing and XCTest in the same target

## Test Types to Include

**Unit Tests** (Swift Testing - @Suite/@Test):
- Happy path scenarios
- Edge cases (empty, nil, max values)
- Error conditions
- Boundary values

**Integration Tests** (Swift Testing):
- Repository with Cache
- UseCase with multiple dependencies
- ViewModel with real UseCases (mocked network)

**UI Tests** (XCUITest - separate target):
- Critical user flows
- Multi-step processes
- Use `/xcuitest` command for these

## Coverage Requirements

- **80% minimum** for all code
- **100% required** for:
  - Financial calculations
  - Authentication logic
  - Security-critical code (Keychain, encryption)
  - Core business logic (UseCases)

## Test Framework Selection

| Project | Framework | Notes |
|---------|-----------|-------|
| iOS 17+ new project | Swift Testing | Recommended |
| iOS 16 or below | XCTest | Required for compatibility |
| UI Tests | XCUITest | Separate target, always XCTest |
| Mixed project | Both | Swift Testing for unit, XCTest for UI |

## Parameterized Tests

Swift Testing's killer feature - test multiple inputs with single test:

```swift
@Test(arguments: [
    (volume: 100_000.0, expected: 80.0...100.0),
    (volume: 100.0, expected: 0.0...30.0),
    (volume: 0.0, expected: 0.0...0.0)
])
func calculateScore_withVaryingVolume(volume: Double, expected: ClosedRange<Double>) {
    let market = MarketData(
        totalVolume: volume,
        bidAskSpread: 0.01,
        activeTraders: 100,
        lastTradeTime: Date()
    )

    let score = sut.calculateScore(for: market)

    #expect(expected.contains(score))
}
```

## @MainActor ViewModel Testing

When testing `@MainActor` ViewModels:

```swift
@Suite @MainActor struct HomeViewModelTests {
    @Test func fetchItems_updatesState() async {
        // Given
        let mockRepo = MockItemRepository()
        mockRepo.items = [Item.stub()]
        let sut = HomeViewModel(repository: mockRepo)

        // When
        await sut.fetchItems()

        // Then
        #expect(sut.items.count == 1)
        #expect(!sut.isLoading)
    }
}
```

**Note**: Apply `@MainActor` to the entire `@Suite` when testing main-actor-isolated types.

## Running Tests

### Xcode Project

```bash
# Run all tests
xcodebuild test \
  -scheme MyApp \
  -destination 'platform=iOS Simulator,name=iPhone 16'

# Run specific test suite
xcodebuild test \
  -scheme MyApp \
  -destination 'platform=iOS Simulator,name=iPhone 16' \
  -only-testing:MyAppTests/LiquidityCalculatorTests

# With coverage
xcodebuild test \
  -scheme MyApp \
  -destination 'platform=iOS Simulator,name=iPhone 16' \
  -enableCodeCoverage YES
```

### Swift Package

```bash
# Run all tests
swift test

# Run specific test
swift test --filter LiquidityCalculatorTests

# With parallel execution
swift test --parallel
```

## CI/CD Integration

Add to your workflow for automated TDD verification:

```yaml
# GitHub Actions example
- name: Run Tests
  run: |
    xcodebuild test \
      -scheme MyApp \
      -destination 'platform=iOS Simulator,name=iPhone 16' \
      -enableCodeCoverage YES \
      -resultBundlePath TestResults.xcresult

- name: Check Coverage
  run: |
    xcrun xccov view --report TestResults.xcresult --json > coverage.json
    # Fail if coverage < 80%
```

## Important Notes

**MANDATORY**: Tests must be written BEFORE implementation. The TDD cycle is:

1. **RED** - Write failing test with `#expect`
2. **GREEN** - Implement to pass
3. **REFACTOR** - Improve code

Never skip the RED phase. Never write code before tests.

## Integration with Other Commands

- Use `/plan` first to understand what to build
- Use `/swift-tdd` to implement with tests
- Use `/xcode-build-fix` if build errors occur
- Use `/swift-review` to review implementation
- Use `/ios-coverage` to verify coverage

## Related Agents

This command invokes the `swift-tdd-guide` agent located at:
`~/.claude/agents/swift-tdd-guide.md`
