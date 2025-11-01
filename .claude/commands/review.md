---
description: Review uncommitted code changes before committing
allowed-tools: Bash, Task, Read
---

Review all uncommitted changes in the repository:

1. Run `git status` to see modified files
2. Run `git diff` to see the actual changes
3. For frontend changes (*.tsx, *.ts files): Use the frontend-reviewer subagent
4. For backend changes (*.py files): Use the backend-reviewer subagent
5. Provide a summary of all changes and recommendations

Focus on:
- Code quality and consistency
- Potential bugs or issues
- Security considerations
- Performance implications
- Testing requirements

End with a recommendation: READY TO COMMIT or NEEDS CHANGES.
