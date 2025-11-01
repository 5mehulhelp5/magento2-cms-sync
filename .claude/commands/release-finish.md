---
description: Finish a release branch and merge to main and develop
allowed-tools: Bash, Task
---

Finish release branch and prepare to merge to main and develop.

## Process

1. **Verify release readiness**
2. **Review all changes**
3. **Create PRs to main and develop**
4. **Tag release**

## Commands

```bash
# Get current branch
current_branch=$(git branch --show-current)

# Verify it's a release branch
if [[ ! "$current_branch" =~ ^release/ ]]; then
  echo "❌ Error: Not on a release branch"
  echo "Current branch: $current_branch"
  echo ""
  echo "Release branches must be named: release/v*"
  echo "To start a release: /release-start <version>"
  exit 1
fi

# Extract version from branch name
version="${current_branch#release/}"

echo "🚀 Finishing release: $version"
echo ""

# Check for uncommitted changes
if ! git diff-index --quiet HEAD --; then
  echo "⚠️  Warning: Uncommitted changes detected"
  git status --short
  echo ""
  echo "Please commit all changes first."
  exit 1
fi

# Update from develop and main
echo "📥 Fetching latest branches..."
git fetch origin develop main

echo ""
echo "🧪 Running final checks..."
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
```

Now use pr-reviewer subagent for release review:

**RELEASE REVIEW - Version $version**

Review this release branch before merging to production:

Branch: `$current_branch`
Target: `main` (then → `develop`)

**Release-Specific Checks**:
1. ✅ Version numbers updated everywhere
2. ✅ CHANGELOG.md updated with all changes
3. ✅ No new features (only bug fixes)
4. ✅ All tests pass
5. ✅ Documentation updated
6. ✅ Migration scripts included (if needed)
7. ✅ Breaking changes documented

**Review Checklist**:
- Git Flow compliance
- Code quality of any bug fixes
- Security review
- Performance impact
- Deployment readiness

After review, if APPROVED:

```bash
# Push release branch
echo "📤 Pushing release branch..."
git push -u origin $current_branch

echo ""
echo "✅ RELEASE READY FOR PRODUCTION"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo ""
echo "Version: $version"
echo ""
echo "STEP 1: Merge to main (production)"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "1. Create PR to main:"
echo "   gh pr create --base main --head $current_branch \\"
echo "     --title \"Release $version\" \\"
echo "     --body \"Production release $version\""
echo ""
echo "2. Get PR approved by team"
echo "3. Merge to main (use squash or merge commit)"
echo ""
echo "4. Tag the release on main:"
echo "   git checkout main"
echo "   git pull origin main"
echo "   git tag -a $version -m \"Release $version\""
echo "   git push origin $version"
echo ""
echo "5. Deploy to production"
echo ""
echo "STEP 2: Merge back to develop"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "After main merge and deployment:"
echo "1. Create PR to develop:"
echo "   gh pr create --base develop --head $current_branch \\"
echo "     --title \"Merge release $version to develop\" \\"
echo "     --body \"Bringing release changes back to develop\""
echo ""
echo "2. Merge to develop"
echo "3. Delete release branch"
echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "📋 Release Deployment Checklist:"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "  [ ] All tests passed"
echo "  [ ] Documentation reviewed"
echo "  [ ] CHANGELOG.md complete"
echo "  [ ] PR to main created and approved"
echo "  [ ] Merged to main"
echo "  [ ] Release tagged: $version"
echo "  [ ] Deployed to production"
echo "  [ ] Production smoke tests passed"
echo "  [ ] Monitoring checked (no errors)"
echo "  [ ] PR to develop created"
echo "  [ ] Merged to develop"
echo "  [ ] Release branch deleted"
echo "  [ ] Release notes published"
echo "  [ ] Team and users notified"
echo ""
echo "🎉 Ready to ship $version!"
```

If NEEDS CHANGES:

List all issues that must be resolved before production release.
