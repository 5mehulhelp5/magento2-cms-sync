---
description: Finish a feature branch and prepare for merge to develop
allowed-tools: Bash, Task, Read
---

Finish the current feature branch and prepare for merge to develop.

## Process

1. **Verify Git Flow compliance**:
   - Ensure on a feature/* branch
   - Check all changes committed
   - Verify tests pass

2. **Update from develop**:
   - Fetch latest develop
   - Rebase on develop (avoid merge commits)

3. **Run comprehensive review**:
   - Use pr-reviewer subagent
   - Check code quality
   - Verify tests pass
   - Check types

4. **Push and create PR**:
   - Push feature branch
   - Create PR to develop
   - Provide PR template

## Commands

```bash
# Get current branch
current_branch=$(git branch --show-current)

# Verify it's a feature branch
if [[ ! "$current_branch" =~ ^feature/ ]] && [[ ! "$current_branch" =~ ^bugfix/ ]]; then
  echo "❌ Error: Not on a feature or bugfix branch"
  echo "Current branch: $current_branch"
  echo ""
  echo "Git Flow feature branches should be named:"
  echo "  feature/<name> or bugfix/<name>"
  exit 1
fi

echo "🔍 Finishing branch: $current_branch"
echo ""

# Check for uncommitted changes
if ! git diff-index --quiet HEAD --; then
  echo "⚠️  Warning: You have uncommitted changes"
  git status --short
  echo ""
  echo "Please commit or stash your changes first."
  exit 1
fi

# Fetch latest develop
echo "📥 Fetching latest develop..."
git fetch origin develop

# Check if up-to-date with develop
merge_base=$(git merge-base HEAD origin/develop)
develop_head=$(git rev-parse origin/develop)

if [ "$merge_base" != "$develop_head" ]; then
  echo "⚠️  Your branch is behind develop. Rebasing..."
  echo ""

  # Rebase on develop
  if git rebase origin/develop; then
    echo "✅ Rebased successfully on develop"
  else
    echo "❌ Rebase conflicts detected"
    echo ""
    echo "Please resolve conflicts manually:"
    echo "  1. Fix conflicts in the files shown above"
    echo "  2. git add <resolved-files>"
    echo "  3. git rebase --continue"
    echo "  4. Run /feature-finish again"
    exit 1
  fi
else
  echo "✅ Already up-to-date with develop"
fi

echo ""
echo "🧪 Running tests and checks..."
```

After the bash commands, continue with:

Now use the pr-reviewer subagent to perform a comprehensive review:

**Review this branch before merging to develop:**
- Branch: `$current_branch` → `develop`
- Review all changes since branching from develop
- Check Git Flow compliance
- Verify code quality
- Ensure tests pass
- Check for security issues

After review, provide:

1. **PR Creation Command**:
```bash
# Push branch to remote
git push -u origin $current_branch

# Create PR using GitHub CLI (if available)
gh pr create --base develop --head $current_branch --title "Title" --body "Description"

# Or provide manual PR creation link
```

2. **PR Template**:
```markdown
## Description
[Describe what this feature does]

## Type of Change
- [ ] New feature
- [ ] Bug fix
- [ ] Documentation update
- [ ] Refactoring

## Testing
- [ ] Tests added/updated
- [ ] All tests pass locally
- [ ] Type checking passes

## Checklist
- [ ] Code follows project conventions
- [ ] Documentation updated
- [ ] No breaking changes (or documented)
- [ ] Ready for review

## Related Issues
Closes #[issue number]
```

3. **Next Steps**:
```
✅ Branch ready for review

Next steps:
  1. Create PR: $current_branch → develop
  2. Wait for code review approval
  3. Address any feedback
  4. Merge PR when approved
  5. Delete feature branch after merge

🔗 PR Review Process:
  - Reviewer will use /pr-review command
  - Address all "Critical" and "Important" feedback
  - Re-request review after changes
  - Squash and merge when approved
```

## If not on a feature branch

Show error and guidance:

```
❌ Not on a feature or bugfix branch

Current branch: [branch-name]

Git Flow feature branches must be named:
  feature/<name> or bugfix/<name>

To start a new feature:
  /feature-start <feature-name>
```
