# iOS Git Workflow

## Commit Message Format

```
<type>: <description>

<optional body>
```

Types: feat, fix, refactor, docs, test, chore, perf, ci

## Pull Request Workflow

When creating PRs:
1. Analyze full commit history (not just latest commit)
2. Use `git diff [base-branch]...HEAD` to see all changes
3. Draft comprehensive PR summary
4. Include test plan with TODOs
5. Push with `-u` flag if new branch

## Feature Implementation Workflow

1. **Plan First**
   - Use **ios-planner** agent to create implementation plan
   - Identify dependencies and risks
   - Break down into phases

2. **TDD Approach**
   - Use **swift-tdd-guide** agent
   - Write tests first (RED)
   - Implement to pass tests (GREEN)
   - Refactor (IMPROVE)
   - Verify 80%+ coverage

3. **Code Review**
   - Use **swift-code-reviewer** agent immediately after writing code
   - Address CRITICAL and HIGH issues
   - Fix MEDIUM issues when possible

4. **Commit & Push**
   - Detailed commit messages
   - Follow conventional commits format

---

## iOS-Specific Git Considerations

### Project File Conflicts (*.pbxproj)

The `project.pbxproj` file is the most conflict-prone file in iOS projects.

**Prevention Strategies:**
- Communicate with team before adding/removing files
- Use feature branches and merge frequently
- Consider tools like [mergepbx](https://github.com/aspect-build/rules_xcodeproj) or Tuist

**Resolution Steps:**
1. Keep both changes if adding different files
2. For UUID conflicts, regenerate the conflicting entry
3. When in doubt: `git checkout --theirs`, then manually re-add your files in Xcode

```bash
# Check pbxproj syntax after manual merge
plutil -lint Project.xcodeproj/project.pbxproj
```

### Asset Catalogs (*.xcassets)

**Best Practices:**
- Each asset should be added in a separate commit
- Use descriptive names: `icon-profile-placeholder` not `icon1`
- Avoid modifying Contents.json manually

**Recommended .gitattributes:**
```gitattributes
*.pbxproj merge=union
*.xcassets -diff
*.png binary
*.jpg binary
```

### Storyboard and XIB Files

**Conflict Prevention:**
- Prefer SwiftUI for new UI development
- One screen per Storyboard file (if using Storyboards)
- Avoid concurrent edits to same UI file

**Resolution:**
- Often easier to discard one version and recreate changes
- Use Xcode's source control comparison for visual diff

### Dependency Lock Files

| Package Manager | Lock File | Action |
|-----------------|-----------|--------|
| SPM | Package.resolved | Always commit, regenerate on conflict |
| CocoaPods | Podfile.lock | Always commit, run `pod install` on conflict |
| Carthage | Cartfile.resolved | Always commit |

```bash
# SPM: Regenerate Package.resolved
rm Package.resolved
swift package resolve

# CocoaPods: Regenerate Podfile.lock
rm Podfile.lock
pod install
```

### Info.plist and Build Settings

**Version/Build Number Conflicts:**
- Use CI/CD to auto-increment build numbers
- Or use build scripts with `agvtool`

```bash
# Increment build number (avoids manual plist edits)
agvtool next-version -all
```

**Sensitive Keys:**
- Never commit API keys in Info.plist
- Use `.xcconfig` files with `.gitignore` for secrets

### Recommended .gitignore

```gitignore
# Xcode User Data
*.xcuserstate
xcuserdata/
*.xcscmblueprint

# Build Products
build/
DerivedData/
*.ipa
*.dSYM.zip
*.dSYM

# Dependencies (if not checking in)
# Pods/
# Carthage/Build/

# Secrets
*.xcconfig
!*.xcconfig.template

# SwiftPM
.swiftpm/xcode/package.xcworkspace/

# Fastlane
fastlane/report.xml
fastlane/Preview.html
fastlane/screenshots/**/*.png

# Code Injection
.inject.entries.txt
```

### Branch Strategy for iOS

**Recommended Flow:**
```
main (production)
  └── develop (integration)
        ├── feature/ABC-123-feature-name
        ├── bugfix/ABC-456-bug-description
        └── release/1.2.0
```

**Release Branch Checklist:**
- [ ] Version bump (CFBundleShortVersionString)
- [ ] Build number increment (CFBundleVersion)
- [ ] Update CHANGELOG.md
- [ ] Privacy manifest review
- [ ] App Store metadata update

### Pre-commit Hooks for iOS

**Useful hooks:**
```bash
#!/bin/bash
# .git/hooks/pre-commit

# Validate pbxproj
plutil -lint *.xcodeproj/project.pbxproj || exit 1

# Run SwiftLint
if which swiftlint > /dev/null; then
  swiftlint lint --quiet
fi

# Check for secrets
if git diff --cached --name-only | xargs grep -l "API_KEY\|SECRET" 2>/dev/null; then
  echo "ERROR: Possible secret detected"
  exit 1
fi
```
