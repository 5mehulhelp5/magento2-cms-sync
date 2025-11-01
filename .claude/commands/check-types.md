---
description: Check TypeScript types and Python type hints
allowed-tools: Bash, Read
---

Verify type safety across the codebase:

1. **Frontend TypeScript**:
   ```bash
   cd frontend && npx tsc --noEmit
   ```
   - Check for type errors
   - Verify all interfaces are properly defined
   - Look for any `any` types that should be fixed

2. **Backend Python**:
   ```bash
   cd backend && mypy . --strict
   ```
   - Check type hints coverage
   - Verify Pydantic schema consistency
   - Check SQLAlchemy model annotations

3. **Cross-stack Type Consistency**:
   - Compare `frontend/src/types/index.ts` with `backend/models/schemas.py`
   - Ensure API request/response types match
   - Flag any mismatches

Report:
- Total type errors found
- Critical type safety issues
- Suggested fixes with code examples
- Files that need type annotations
