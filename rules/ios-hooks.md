# iOS Hooks System

## Hook Types

- **PreToolUse**: Before tool execution (validation, parameter modification)
- **PostToolUse**: After tool execution (auto-format, checks)
- **PreCompact**: Before context compaction (save state)
- **SessionStart**: When session begins (load context)
- **Stop**: When session ends (final verification)

## Current Hooks (in ~/.claude/settings.json)

### PreToolUse
- **git push review**: Shows diff for review before push
- **git secrets prevention**: Blocks commits containing API keys (AKIA, sk-, ghp_)
- **certificate blocker**: Prevents committing .mobileprovision, .p12, .cer files
- **doc blocker**: Blocks creation of unnecessary .md/.txt files

### PostToolUse
- **SwiftFormat**: Auto-formats Swift files after edit
- **SwiftLint**: Runs linting on edited Swift files
- **Swift syntax check**: Runs `swiftc -parse` (faster than xcodebuild)
- **print() warning**: Warns about print() in edited files (use os.Logger instead)
- **Memory leak detection**: Warns about `[self]` without `[weak self]`
- **Swift 6 hints**: Suggests @MainActor over DispatchQueue.main.async
- **Xcode file validation**: Runs plutil on pbxproj, plist, strings, entitlements
- **PR creation**: Logs PR URL after gh pr create

### Stop
- **print() audit**: Checks all modified Swift files for print()
- **Secrets audit**: Reviews staged files for hardcoded secrets

## Auto-Accept Permissions

Use with caution:
- Enable for trusted, well-defined plans
- Disable for exploratory work
- Never use `dangerouslySkipPermissions` flag
- Configure `allowedTools` in `~/.claude.json` instead

**Safe to auto-accept**: Read, Glob, Grep, swiftformat, swiftlint, git status/diff/log

**Never auto-accept**: xcodebuild, git push, fastlane, rm, Write(*.swift)

## TodoWrite Best Practices

Use TodoWrite tool to:
- Track progress on multi-step tasks
- Verify understanding of instructions
- Enable real-time steering
- Show granular implementation steps

Good granularity for iOS:
- "Create UserModel.swift with Codable"
- "Add UserRepository protocol"
- "Write unit tests for UserRepository"

Bad granularity:
- "Implement user feature"
- "Add tests"

