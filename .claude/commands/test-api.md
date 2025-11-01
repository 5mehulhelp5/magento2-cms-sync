---
description: Test Magento API connection for an instance
argument-hint: <instance_id>
allowed-tools: Read, Bash
---

Test the Magento API connection for instance ID: $ARGUMENTS

Steps:
1. Read the instance configuration from the database or data files in `backend/data/instances/$ARGUMENTS/`
2. Extract the base URL and API token
3. Run a curl command to test the connection
4. Test both authentication and CMS endpoints
5. Report the results

Provide clear output showing:
- Connection status (success/failure)
- Response time
- API version if available
- Sample data (first 2 blocks/pages)
- Any errors encountered

If instance ID is not provided, list available instances first.
