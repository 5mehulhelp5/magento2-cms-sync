---
description: Start a release branch for preparing a new version (Git Flow)
argument-hint: <version>
allowed-tools: Bash
---

Start a release branch for version: $ARGUMENTS

## Process

1. **Create release branch**:
   - Name: `release/$ARGUMENTS`
   - Branch from: `develop`
   - Purpose: Prepare for production release

2. **Provide release checklist**

## Commands

```bash
# Verify version provided
version="$ARGUMENTS"
if [ -z "$version" ]; then
  echo "❌ Error: Version number required"
  echo ""
  echo "Usage: /release-start <version>"
  echo ""
  echo "Examples:"
  echo "  /release-start v1.2.0"
  echo "  /release-start 1.2.0"
  exit 1
fi

# Ensure version starts with 'v' or add it
if [[ ! "$version" =~ ^v ]]; then
  version="v$version"
fi

# Check current state
echo "📋 Checking repository state..."
git status

# Switch to develop
current_branch=$(git branch --show-current)
if [ "$current_branch" != "develop" ]; then
  echo "📍 Switching to develop branch..."
  git checkout develop
fi

# Update develop
echo "📥 Pulling latest develop..."
git pull origin develop

# Verify clean working directory
if ! git diff-index --quiet HEAD --; then
  echo "⚠️  Warning: Uncommitted changes detected"
  git status --short
  echo ""
  echo "Please commit or stash changes first."
  exit 1
fi

# Create release branch
branch_name="release/$version"
echo ""
echo "🚀 Creating release branch: $branch_name"
git checkout -b "$branch_name"

echo ""
echo "✅ Release branch created successfully!"
echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "📋 RELEASE PREPARATION CHECKLIST"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo ""
echo "1️⃣  UPDATE VERSION NUMBERS:"
echo "   [ ] Update package.json (frontend)"
echo "   [ ] Update version in backend/__init__.py or config"
echo "   [ ] Update version in CHANGELOG.md"
echo ""
echo "2️⃣  FINAL TESTING:"
echo "   [ ] Run full test suite: /run-tests all"
echo "   [ ] Check types: /check-types"
echo "   [ ] Manual testing of critical features"
echo "   [ ] Test with real Magento instances"
echo ""
echo "3️⃣  DOCUMENTATION:"
echo "   [ ] Update CHANGELOG.md with all changes"
echo "   [ ] Review and update README.md if needed"
echo "   [ ] Update API documentation"
echo "   [ ] Check all documentation links work"
echo ""
echo "4️⃣  BUG FIXES ONLY:"
echo "   [ ] Fix any last-minute bugs found in testing"
echo "   [ ] NO NEW FEATURES on release branch"
echo "   [ ] Keep changes minimal"
echo ""
echo "5️⃣  FINISH RELEASE:"
echo "   [ ] Use: /release-finish"
echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "⚠️  RELEASE BRANCH RULES:"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "• Branch from: develop"
echo "• Merge to: BOTH main AND develop"
echo "• Only bug fixes and version updates allowed"
echo "• NO new features"
echo "• Thoroughly test before finishing"
echo ""
echo "Current branch: $branch_name"
echo "Target version: $version"
```
