---
description: Start a new feature branch following Git Flow
argument-hint: <feature-name>
allowed-tools: Bash, Read
---

Start a new feature branch following Git Flow: $ARGUMENTS

## Process

1. **Verify current state**:
   - Check we're on `develop` branch
   - Ensure develop is up-to-date
   - Verify working directory is clean

2. **Create feature branch**:
   - Name: `feature/$ARGUMENTS`
   - Branch from: `develop`

3. **Provide guidance**:
   - Show next steps
   - Explain Git Flow workflow

## Commands

```bash
# Check current branch and status
git status

# Ensure we're on develop
current_branch=$(git branch --show-current)
if [ "$current_branch" != "develop" ]; then
  echo "⚠️  Warning: Not on develop branch. Switching to develop..."
  git checkout develop
fi

# Update develop
echo "📥 Pulling latest changes from develop..."
git pull origin develop

# Create and checkout feature branch
feature_name="$ARGUMENTS"
if [ -z "$feature_name" ]; then
  echo "❌ Error: Feature name required"
  echo "Usage: /feature-start <feature-name>"
  echo "Example: /feature-start add-widget-sync"
  exit 1
fi

branch_name="feature/$feature_name"
echo "🌿 Creating feature branch: $branch_name"
git checkout -b "$branch_name"

echo ""
echo "✅ Feature branch created successfully!"
echo ""
echo "📋 Next steps:"
echo "  1. Make your changes"
echo "  2. Commit regularly: git add . && git commit -m 'description'"
echo "  3. Push to remote: git push -u origin $branch_name"
echo "  4. When done, use: /feature-finish"
echo ""
echo "💡 Remember:"
echo "  - Keep commits focused and atomic"
echo "  - Write clear commit messages"
echo "  - Add tests for new functionality"
echo "  - Update documentation as needed"
echo ""
echo "🔍 Review your code before finishing:"
echo "  /review - Review uncommitted changes"
echo "  /run-tests all - Run all tests"
echo "  /check-types - Verify type safety"
```

## If feature name not provided

If $ARGUMENTS is empty, show usage:

```
❌ Feature name required

Usage: /feature-start <feature-name>

Examples:
  /feature-start add-widget-sync
  /feature-start TICKET-123-improve-comparison
  /feature-start bugfix/fix-sync-error

📖 Git Flow feature workflow:
  1. /feature-start <name>    - Start feature (branches from develop)
  2. Make changes & commit
  3. /feature-finish          - Merge to develop via PR
```
