---
description: Review a pull request before merging (Git Flow compliant)
argument-hint: [source-branch] [target-branch]
allowed-tools: Bash, Task, Read, Grep
---

Review pull request before merging: $ARGUMENTS

## Process

1. **Determine branches**:
   - If arguments provided: use those branches
   - If no arguments: use current branch → develop

2. **Use pr-reviewer subagent**:
   - Comprehensive Git Flow compliance check
   - Code quality review
   - Security analysis
   - Merge safety verification

3. **Provide detailed report**:
   - Git Flow compliance status
   - Code quality issues
   - Test results
   - Merge recommendation

## Commands

```bash
# Parse arguments or use defaults
if [ -z "$ARGUMENTS" ]; then
  # No arguments - use current branch
  source_branch=$(git branch --show-current)

  # Determine target based on Git Flow
  if [[ "$source_branch" =~ ^feature/ ]] || [[ "$source_branch" =~ ^bugfix/ ]]; then
    target_branch="develop"
  elif [[ "$source_branch" =~ ^release/ ]]; then
    target_branch="main"  # Release goes to main first
  elif [[ "$source_branch" =~ ^hotfix/ ]]; then
    target_branch="main"  # Hotfix goes to main first
  else
    target_branch="develop"
  fi

  echo "📋 Reviewing: $source_branch → $target_branch"
else
  # Arguments provided
  read source_branch target_branch <<< "$ARGUMENTS"
  if [ -z "$target_branch" ]; then
    target_branch="develop"
  fi
  echo "📋 Reviewing: $source_branch → $target_branch"
fi

# Verify branches exist
if ! git rev-parse --verify "$source_branch" >/dev/null 2>&1; then
  echo "❌ Error: Source branch '$source_branch' does not exist"
  exit 1
fi

if ! git rev-parse --verify "$target_branch" >/dev/null 2>&1; then
  echo "❌ Error: Target branch '$target_branch' does not exist"
  exit 1
fi

# Get branch info
echo ""
echo "🔍 Branch Information:"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
git log --oneline --graph "$target_branch..$source_branch" | head -20
echo ""

# Show diff summary
echo "📊 Changes Summary:"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
git diff --stat "$target_branch...$source_branch"
echo ""

# List changed files
echo "📝 Files Changed:"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
git diff --name-status "$target_branch...$source_branch"
echo ""

# Check for merge conflicts
echo "🔀 Checking for merge conflicts..."
merge_check=$(git merge-tree $(git merge-base "$target_branch" "$source_branch") "$target_branch" "$source_branch" | grep -c "^+<<<<<<< " || echo "0")

if [ "$merge_check" -gt 0 ]; then
  echo "⚠️  WARNING: Merge conflicts detected!"
else
  echo "✅ No merge conflicts detected"
fi
echo ""
```

After gathering information, use the pr-reviewer subagent:

**Comprehensive PR Review Request:**

Review the pull request for merging `$source_branch` into `$target_branch`:

1. **Git Flow Compliance**:
   - Verify branch naming convention
   - Check source and target are appropriate
   - Ensure no direct commits to develop/main

2. **Code Quality**:
   - Review all changed files
   - Use frontend-reviewer for .ts/.tsx files
   - Use backend-reviewer for .py files
   - Check for code smells and duplication

3. **Testing**:
   - Verify tests exist for new features
   - Check test coverage
   - Run tests if possible

4. **Security**:
   - Check for hardcoded secrets
   - Verify input validation
   - Look for security vulnerabilities

5. **Documentation**:
   - Ensure CLAUDE.md updated if needed
   - Check README.md for user-facing changes
   - Verify code comments for complex logic

Provide a complete review report with:
- Git Flow compliance status
- Issues found (Critical/Important/Suggestions)
- Merge recommendation (APPROVE/REQUEST CHANGES/NEEDS DISCUSSION)
- Action items for the developer

## If no arguments and not on a branch

Show usage:

```
Usage: /pr-review [source-branch] [target-branch]

Examples:
  /pr-review                              # Review current branch → develop
  /pr-review feature/new-feature develop  # Review specific branch
  /pr-review hotfix/critical-fix main     # Review hotfix

💡 The command will automatically:
  - Detect Git Flow branch types
  - Determine appropriate target branch
  - Run comprehensive review
  - Check for conflicts
  - Verify compliance
```
