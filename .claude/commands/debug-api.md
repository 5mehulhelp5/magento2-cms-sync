---
description: Debug Magento API integration issues
argument-hint: <instance_id or error_description>
allowed-tools: Read, Bash, Task, Grep
---

Debug Magento API issues: $ARGUMENTS

Use the magento-api-debugger subagent to:
1. Analyze the error or issue described
2. Check instance configuration
3. Test API connectivity
4. Review integration code in `backend/integrations/magento_client.py`
5. Examine cached data and logs
6. Provide specific fix recommendations

If an instance ID is provided, focus on that specific instance.
If an error description is provided, diagnose the root cause.

Output should include:
- Root cause analysis
- Recommended fix
- Test commands to verify
- Code changes if needed
