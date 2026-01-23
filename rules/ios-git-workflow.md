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
# Disable merge=union for pbxproj - can cause UUID duplicates
# Manual merge is safer than automatic union merge
*.pbxproj -merge
*.xcassets -diff
*.png binary
*.jpg binary

# Git LFS for large files
*.mp4 filter=lfs diff=lfs merge=lfs -text
*.mov filter=lfs diff=lfs merge=lfs -text
*.zip filter=lfs diff=lfs merge=lfs -text
```

### Git LFS for Large Files

For assets over 50MB, use Git LFS to avoid repository bloat.

**Setup:**
```bash
git lfs install
git lfs track "*.mp4"
git lfs track "*.mov"
git lfs track "*.zip"
git lfs track "Resources/**/*.png"  # Large image directories
```

**Verify LFS tracking:**
```bash
git lfs ls-files
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

### Signing & Provisioning

**NEVER commit these files:**
- `*.p12` (certificates)
- `*.mobileprovision` (provisioning profiles)
- `*.cer` (certificates)

**Recommended approach: Fastlane Match**
```bash
# Setup match for team signing
fastlane match init
fastlane match development
fastlane match appstore
```

**XCConfig Hierarchy for Secrets:**
```
Config/
├── Base.xcconfig           # Shared settings
├── Debug.xcconfig          # imports Base, Debug-specific
├── Release.xcconfig        # imports Base, Release-specific
├── Secrets.xcconfig        # API keys (gitignored)
└── Secrets.xcconfig.template  # Template for team (committed)
```

```xcconfig
// Secrets.xcconfig.template (committed)
API_BASE_URL = https://api.example.com
API_KEY = YOUR_API_KEY_HERE

// Secrets.xcconfig (gitignored, actual values)
API_BASE_URL = https://api.example.com
API_KEY = sk-actual-key-here
```

### Xcode Cloud

For Xcode Cloud CI/CD, use the `ci_scripts/` folder:

```
ci_scripts/
├── ci_post_clone.sh    # After repo clone (install dependencies)
├── ci_pre_xcodebuild.sh  # Before build (setup secrets)
└── ci_post_xcodebuild.sh # After build (notifications, uploads)
```

**Example ci_post_clone.sh:**
```bash
#!/bin/bash
set -e

# Install dependencies
cd "$CI_PRIMARY_REPOSITORY_PATH"

# SPM dependencies (automatic, but can customize)
# swift package resolve

# CocoaPods (if used)
# brew install cocoapods
# pod install

# Create Secrets.xcconfig from environment variables
cat > "Config/Secrets.xcconfig" << EOF
API_KEY = $API_KEY
API_BASE_URL = $API_BASE_URL
EOF
```

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

# Check for secrets (comprehensive patterns)
SECRET_PATTERNS="API_KEY|SECRET|PASSWORD|PRIVATE_KEY|ACCESS_TOKEN|Bearer|sk-|pk_live|pk_test|-----BEGIN"
if git diff --cached --name-only | xargs grep -lE "$SECRET_PATTERNS" 2>/dev/null; then
  echo "ERROR: Possible secret detected in staged files"
  echo "If intentional (e.g., template file), use: git commit --no-verify"
  exit 1
fi

# Check for large files (>5MB)
LARGE_FILES=$(git diff --cached --name-only | xargs -I{} sh -c 'test -f "{}" && du -k "{}" | awk "\$1 > 5000 {print \$2}"')
if [ -n "$LARGE_FILES" ]; then
  echo "WARNING: Large files detected (>5MB). Consider Git LFS:"
  echo "$LARGE_FILES"
fi
```

### Build Phase Scripts

When using Run Script phases in Xcode, always specify input/output files for incremental builds:

```bash
# Good: Xcode skips script if inputs unchanged
Input Files:
  $(SRCROOT)/scripts/version.sh
  $(SRCROOT)/Config/Base.xcconfig
Output Files:
  $(DERIVED_FILE_DIR)/version-generated.swift

# Bad: Script runs on every build (no input/output files)
```
