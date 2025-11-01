# Claude Code Quick Reference - Magento CMS Sync

## 🌿 Git Flow Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `/feature-start <name>` | Start feature branch | `/feature-start add-widgets` |
| `/feature-finish` | Finish & review feature | `/feature-finish` |
| `/pr-review [src] [tgt]` | **Review before merge** | `/pr-review` |
| `/hotfix-start <name>` | Start production hotfix | `/hotfix-start critical-fix` |
| `/hotfix-finish` | Finish hotfix | `/hotfix-finish` |
| `/release-start <ver>` | Start release | `/release-start v1.2.0` |
| `/release-finish` | Finish release | `/release-finish` |

## 🚀 Development Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `/review` | Review uncommitted changes | `/review` |
| `/run-tests <target>` | Run tests | `/run-tests all` |
| `/check-types` | Type safety check | `/check-types` |
| `/add-test <file>` | Generate tests | `/add-test backend/services/sync.py` |

## 🔧 Magento Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `/test-api <id>` | Test Magento API | `/test-api 1` |
| `/debug-api <id>` | Debug API issues | `/debug-api 1` |
| `/check-sync` | Validate sync safety | `/check-sync` |
| `/refresh-data <id>` | Refresh cache | `/refresh-data all` |

## 🤖 Subagents

| Agent | Auto-triggered on | Manual invocation |
|-------|-------------------|-------------------|
| `pr-reviewer` | **PR review (REQUIRED)** | Automatic with `/pr-review` |
| `magento-api-debugger` | API errors | "Debug this Magento API error" |
| `sync-validator` | Sync operations | "Validate this sync operation" |
| `frontend-reviewer` | Frontend changes | "Review this React component" |
| `backend-reviewer` | Backend changes | "Review this FastAPI endpoint" |
| `test-generator` | Never (manual) | "Generate tests for this file" |

## 📁 Project Structure

```
backend/
├── api/          # FastAPI routes (thin)
├── services/     # Business logic (thick)
├── models/       # DB models + schemas
└── integrations/ # Magento client

frontend/src/
├── pages/        # Route components
├── components/   # Reusable UI
├── services/     # API clients
├── store/        # Zustand state
└── types/        # TypeScript types
```

## 🔧 Common Tasks

### Start Development
```bash
./start-dev.sh
```

### Git Flow: New Feature
```bash
/feature-start my-feature    # Start from develop
# ... make changes ...
/review                      # Review changes
/run-tests all              # Run tests
git commit -m "message"     # Commit
/feature-finish             # Review & create PR
# Merge PR after approval
```

### Git Flow: Hotfix
```bash
/hotfix-start critical-fix  # Start from main
# ... fix issue ...
/run-tests all
/hotfix-finish             # Merge to main + develop
```

### Before Committing
```bash
/review
/run-tests all
/check-types
```

### Adding a Feature
1. `/feature-start <name>` - Start feature branch
2. Read CLAUDE.md for context
3. Follow service layer pattern
4. Add types/schemas
5. Generate tests with `/add-test`
6. `/feature-finish` - Review and create PR

### Code Review (REQUIRED)
```bash
/pr-review  # Before merging any PR
```

### Debugging API
1. Use `/test-api <id>` first
2. Then `/debug-api <id>` if issues
3. Check `backend/data/instances/<id>/`

## 🌿 Git Flow Rules

1. **Never** commit directly to `main` or `develop`
2. **Always** use feature branches: `/feature-start <name>`
3. **Always** review before merge: `/pr-review`
4. **Always** run tests: `/run-tests all`
5. Features → develop, Releases/Hotfixes → main + develop

## 💡 Tips

- Use `/pr-review` before EVERY merge (REQUIRED)
- Use subagents for complex analysis
- Use slash commands for quick tasks
- Always review before committing
- Generate tests early
- Check types frequently
- Follow Git Flow workflow

## 📚 Full Documentation

- `CLAUDE.md` - AI development guide
- `DEVELOPMENT_WORKFLOW.md` - Detailed workflows
- `IMPROVEMENTS_SUMMARY.md` - What's new
- `README.md` - User guide
