---
description: Run tests for backend and/or frontend
argument-hint: [backend|frontend|all]
allowed-tools: Bash
---

Run tests for: $ARGUMENTS

Commands to execute:

**Backend tests** (if "backend" or "all"):
```bash
cd backend
source venv/bin/activate
pytest -v --cov=. --cov-report=term-missing
```

**Frontend tests** (if "frontend" or "all"):
```bash
cd frontend
npm test -- --coverage --watchAll=false
```

**Type checking**:
```bash
# Backend
cd backend && mypy .

# Frontend
cd frontend && npm run type-check
```

Default to "all" if no argument provided.

Report:
- Test results (pass/fail counts)
- Coverage percentages
- Type errors if any
- Recommendations for fixing failures
