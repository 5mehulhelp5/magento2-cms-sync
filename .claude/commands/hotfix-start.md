---
description: Start a hotfix branch for critical production fixes (Git Flow)
argument-hint: <hotfix-name or version>
allowed-tools: Bash
---

Start a hotfix branch for critical production fix: $ARGUMENTS

## Process

1. **Verify this is a critical fix**:
   - Hotfixes are for production emergencies only
   - For non-critical bugs, use feature/bugfix branches

2. **Create hotfix branch**:
   - Name: `hotfix/$ARGUMENTS`
   - Branch from: `main`
   - Will merge to: `main` AND `develop`

3. **Provide guidance**:
   - Explain hotfix workflow
   - Show next steps

## Commands

```bash
# Verify hotfix name provided
hotfix_name="$ARGUMENTS"
if [ -z "$hotfix_name" ]; then
  echo "❌ Error: Hotfix name or version required"
  echo ""
  echo "Usage: /hotfix-start <name or version>"
  echo ""
  echo "Examples:"
  echo "  /hotfix-start v1.2.4           # Version bump"
  echo "  /hotfix-start critical-sync-fix  # Descriptive name"
  echo ""
  echo "⚠️  Important: Hotfixes are for critical production issues only!"
  echo "   For non-critical bugs, use: /feature-start bugfix/<name>"
  exit 1
fi

# Confirm this is a critical fix
echo "⚠️  HOTFIX WORKFLOW"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "Hotfixes are for CRITICAL production issues only."
echo ""
echo "Hotfix characteristics:"
echo "  ✓ Production is broken or has critical bug"
echo "  ✓ Needs immediate deployment"
echo "  ✓ Cannot wait for next release cycle"
echo ""
echo "For non-critical issues:"
echo "  → Use: /feature-start bugfix/<name>"
echo ""
read -p "Is this truly a critical production fix? (yes/no): " confirm

if [ "$confirm" != "yes" ]; then
  echo "❌ Hotfix cancelled. Use /feature-start for non-critical fixes."
  exit 1
fi

# Check current branch and status
echo ""
echo "📋 Checking repository state..."
git status

# Ensure we're on main
current_branch=$(git branch --show-current)
if [ "$current_branch" != "main" ] && [ "$current_branch" != "master" ]; then
  echo "📍 Switching to main branch..."
  git checkout main || git checkout master
fi

# Update main
echo "📥 Pulling latest changes from main..."
git pull origin main 2>/dev/null || git pull origin master

# Verify working directory is clean
if ! git diff-index --quiet HEAD --; then
  echo "⚠️  Warning: Uncommitted changes detected"
  git status --short
  echo ""
  echo "Please commit or stash changes before starting hotfix."
  exit 1
fi

# Create hotfix branch
branch_name="hotfix/$hotfix_name"
echo ""
echo "🚨 Creating hotfix branch: $branch_name"
git checkout -b "$branch_name"

echo ""
echo "✅ Hotfix branch created successfully!"
echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "📋 HOTFIX WORKFLOW"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo ""
echo "1️⃣  FIX THE ISSUE:"
echo "   - Make minimal changes to fix the critical bug"
echo "   - Add/update tests to prevent regression"
echo "   - Update version number if applicable"
echo ""
echo "2️⃣  TEST THOROUGHLY:"
echo "   /run-tests all    # Run all tests"
echo "   /check-types      # Verify type safety"
echo "   /review           # Review your changes"
echo ""
echo "3️⃣  FINISH HOTFIX:"
echo "   /hotfix-finish    # Merge to main AND develop"
echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "⚠️  IMPORTANT NOTES:"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "• Hotfixes branch from MAIN (not develop)"
echo "• Must merge to BOTH main AND develop"
echo "• Keep changes minimal - only fix the critical issue"
echo "• Tag the release after merging to main"
echo "• Deploy immediately after merge"
echo ""
echo "Current branch: $branch_name"
```
