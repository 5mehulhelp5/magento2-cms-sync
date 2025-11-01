---
description: Finish a hotfix branch and merge to main and develop
allowed-tools: Bash, Task
---

Finish hotfix branch and prepare to merge to main and develop.

## Process

1. **Verify hotfix branch**:
   - Ensure on hotfix/* branch
   - Check all changes committed
   - Run comprehensive review

2. **Prepare for merge to main**:
   - Update from main
   - Run tests
   - Create PR to main

3. **Prepare for merge to develop**:
   - After main merge, also merge to develop

## Commands

```bash
# Get current branch
current_branch=$(git branch --show-current)

# Verify it's a hotfix branch
if [[ ! "$current_branch" =~ ^hotfix/ ]]; then
  echo "❌ Error: Not on a hotfix branch"
  echo "Current branch: $current_branch"
  echo ""
  echo "Hotfix branches must be named: hotfix/<name>"
  echo "To start a hotfix: /hotfix-start <name>"
  exit 1
fi

echo "🚨 Finishing hotfix: $current_branch"
echo ""

# Check for uncommitted changes
if ! git diff-index --quiet HEAD --; then
  echo "⚠️  Warning: Uncommitted changes detected"
  git status --short
  echo ""
  echo "Please commit all changes first."
  exit 1
fi

# Update from main
echo "📥 Fetching latest main..."
git fetch origin main

# Check if up-to-date
merge_base=$(git merge-base HEAD origin/main)
main_head=$(git rev-parse origin/main)

if [ "$merge_base" != "$main_head" ]; then
  echo "⚠️  Your branch is behind main. Rebasing..."

  if git rebase origin/main; then
    echo "✅ Rebased successfully on main"
  else
    echo "❌ Rebase conflicts detected"
    echo "Please resolve conflicts and run /hotfix-finish again"
    exit 1
  fi
else
  echo "✅ Already up-to-date with main"
fi

echo ""
echo "🧪 Running tests and checks..."
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
```

Now use pr-reviewer subagent to review the hotfix:

**HOTFIX REVIEW - CRITICAL**

Review this hotfix before merging to production (main):

Branch: `$current_branch` → `main` (then → `develop`)

**Hotfix-Specific Checks**:
1. ✅ Verify it's truly a critical fix
2. ✅ Changes are minimal (only fix the issue)
3. ✅ No new features added
4. ✅ Tests added/updated
5. ✅ Version bumped (if applicable)
6. ✅ All tests pass
7. ✅ No breaking changes

**Standard Review**:
- Git Flow compliance
- Code quality
- Security
- Documentation

After review, provide:

1. **If APPROVED**:

```bash
# Push hotfix branch
echo "📤 Pushing hotfix branch..."
git push -u origin $current_branch

echo ""
echo "✅ HOTFIX READY FOR MERGE"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo ""
echo "⚠️  IMPORTANT: Hotfixes must merge to BOTH main AND develop"
echo ""
echo "STEP 1: Merge to main (production)"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "1. Create PR: $current_branch → main"
echo "   gh pr create --base main --head $current_branch \\"
echo "     --title \"Hotfix: [description]\" \\"
echo "     --body \"Critical production fix\""
echo ""
echo "2. Get PR approved (code review)"
echo "3. Merge to main"
echo "4. Tag the release:"
echo "   git tag -a v1.x.x -m \"Hotfix release\""
echo "   git push origin v1.x.x"
echo "5. Deploy to production immediately"
echo ""
echo "STEP 2: Merge to develop"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "After main merge:"
echo "1. Create PR: $current_branch → develop"
echo "   gh pr create --base develop --head $current_branch \\"
echo "     --title \"Merge hotfix to develop\" \\"
echo "     --body \"Merging hotfix from main\""
echo ""
echo "2. Merge to develop"
echo "3. Delete hotfix branch"
echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "📋 Hotfix Checklist:"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "  [ ] PR to main created and approved"
echo "  [ ] Merged to main"
echo "  [ ] Release tagged (v1.x.x)"
echo "  [ ] Deployed to production"
echo "  [ ] Production verified working"
echo "  [ ] PR to develop created"
echo "  [ ] Merged to develop"
echo "  [ ] Hotfix branch deleted"
echo "  [ ] Team notified of hotfix"
```

2. **If NEEDS CHANGES**:

List all critical issues that must be fixed before merging to production.

Remind: This is a production hotfix - extra scrutiny required!
