---
description: Refresh cached data for Magento instances
argument-hint: <instance_id or "all">
allowed-tools: Bash, Read
---

Refresh cached Magento data for: $ARGUMENTS

Process:
1. If instance ID provided, refresh that instance
2. If "all" provided, refresh all instances
3. Call the refresh API endpoint: POST /api/compare/refresh/{id}
4. Monitor the refresh progress
5. Verify JSON files are updated in `backend/data/instances/{id}/`

Steps:
```bash
# Check if backend is running
curl http://localhost:8000/ 2>/dev/null || echo "Backend not running. Start with: ./start-dev.sh"

# Refresh instance (requires jq for parsing)
curl -X POST "http://localhost:8000/api/compare/refresh/{instance_id}" \
  -H "Content-Type: application/json" | jq

# Verify data files
ls -lh backend/data/instances/{instance_id}/
```

If backend is not running, provide instructions to start it.

Report:
- Refresh status for each instance
- Number of blocks and pages fetched
- File sizes and timestamps
- Any errors encountered
