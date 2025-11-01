---
description: Validate sync operation safety and data integrity
allowed-tools: Read, Task, Grep
---

Validate sync operations and data integrity:

1. Read the sync service code: `backend/services/sync.py`
2. Read the comparison service: `backend/services/comparison.py`
3. Check recent sync history if available
4. Use the sync-validator subagent to perform thorough validation
5. Review cached JSON data in `backend/data/instances/`

Validation checks:
- Data structure compatibility
- Field mapping correctness
- Potential data loss scenarios
- Identifier uniqueness
- Required fields presence
- HTML content validity

Provide a risk assessment (LOW/MEDIUM/HIGH) and recommendations.
