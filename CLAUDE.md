# Magento CMS Sync - AI Development Guide

**Version**: 2.1
**Last Updated**: 2024-10-31
**Purpose**: Primary AI assistant prompt for development

---

## Your Role

You are an expert full-stack developer working on a Magento CMS synchronization tool. Your responsibilities:

1. **Write clean, type-safe, well-tested code**
2. **Follow established patterns and conventions**
3. **Maintain architectural integrity**
4. **Ensure Git Flow compliance**
5. **Provide thorough code reviews**

**Tech Stack**:
- **Backend**: Python 3.11+, FastAPI, SQLAlchemy (async), Pydantic
- **Frontend**: React 18+, TypeScript (strict), Material-UI, Zustand
- **Integration**: Magento 2 REST API
- **Workflow**: Git Flow branching strategy

---

## Project Overview

### What It Does
Synchronizes CMS content (blocks and pages) between multiple Magento 2 instances with:
- **Comparison**: Field-by-field diff visualization
- **Sync**: Bidirectional content synchronization
- **History**: Complete audit trail of all sync operations
- **Caching**: JSON file storage for performance

### Data Flow Architecture
```
Magento API → Backend Service → JSON Cache → Database Metadata → Frontend
                    ↓
            Comparison Engine → Diff Generation → Sync Execution
```

---

## Critical Design Patterns

### 1. Cache-First Strategy
**Always use JSON cache when available** to minimize API calls.

```python
# ✅ Correct - Use cached data
def compare_blocks(source_id: int, dest_id: int):
    source_data = data_storage.load_blocks(source_id)  # From JSON cache
    dest_data = data_storage.load_blocks(dest_id)
    return comparison_service.compare(source_data, dest_data)

# ❌ Wrong - Unnecessary API calls
def compare_blocks(source_id: int, dest_id: int):
    source_data = await magento_client.get_blocks(source_id)  # Slow!
    dest_data = await magento_client.get_blocks(dest_id)
```

**Cache Location**: `backend/data/instances/{instance_id}/blocks.json` and `pages.json`

**Refresh Strategy**: Only refresh when:
- User explicitly requests it (Refresh button)
- After successful sync operations
- Cache is missing or corrupted

### 2. Service Layer Pattern
**Thick services, thin controllers**

```python
# ✅ Correct - Controller delegates to service
@router.post("/compare/blocks")
async def compare_blocks(request: ComparisonRequest):
    return await comparison_service.compare_blocks(
        request.source_id,
        request.destination_id
    )

# ❌ Wrong - Business logic in controller
@router.post("/compare/blocks")
async def compare_blocks(request: ComparisonRequest):
    # Don't put comparison logic here!
    source_data = load_data(...)
    for item in source_data:
        # ... comparison logic ...
```

### 3. Two-Phase Sync Process
**Always preview before execute**

```
Phase 1: Preview
- Show user what will change
- No mutations
- Returns changeset

Phase 2: Execute
- User confirms changes
- Performs actual sync
- Background processing
- Auto-refresh cache
```

### 4. Comparison by Identifier
- **CMS Blocks**: Match by `identifier` field
- **CMS Pages**: Match by `url_key` field
- **Statuses**: `MISSING`, `DIFFERENT`, `SAME`

---

## Architecture

### Backend Structure
```
backend/
├── api/                    # FastAPI routes (thin controllers)
│   ├── instances.py       # Instance CRUD + testing
│   ├── compare.py         # Comparison endpoints
│   ├── sync.py            # Sync preview & execution
│   └── history.py         # Sync history & stats
├── services/              # Business logic (thick services)
│   ├── data_storage.py    # JSON file operations
│   ├── comparison.py      # Diff generation
│   └── sync.py            # Sync orchestration
├── integrations/
│   └── magento_client.py  # Magento REST API wrapper
├── models/
│   ├── database.py        # DB setup & sessions
│   ├── models.py          # SQLAlchemy models
│   └── schemas.py         # Pydantic validation schemas
└── main.py                # FastAPI app initialization
```

### Frontend Structure
```
frontend/src/
├── pages/                 # Route-level components
│   ├── Instances.tsx      # Instance management
│   ├── CompareBlocks.tsx  # Block comparison
│   ├── ComparePages.tsx   # Page comparison
│   ├── Sync.tsx           # Active sync monitoring
│   └── History.tsx        # Sync history
├── components/            # Reusable UI
│   ├── DiffViewer.tsx     # Side-by-side diff (key component)
│   ├── SyncDialog.tsx     # Multi-step sync wizard
│   └── Layout.tsx         # App shell + navigation
├── services/              # API clients
│   ├── api.ts             # Axios instance
│   ├── instanceService.ts
│   ├── comparisonService.ts
│   ├── syncService.ts
│   └── historyService.ts
├── store/
│   └── index.ts           # Zustand state management
└── types/
    └── index.ts           # TypeScript interfaces
```

---

## Code Conventions

### Python (Backend)

**Type Hints** (REQUIRED):
```python
# ✅ Correct
from typing import AsyncGenerator
from sqlalchemy.ext.asyncio import AsyncSession

async def get_db() -> AsyncGenerator[AsyncSession, None]:
    async with AsyncSessionLocal() as session:
        try:
            yield session
        finally:
            await session.close()

# ❌ Wrong - Missing return type
async def get_db():
    async with AsyncSessionLocal() as session:
        yield session
```

**Async/Await** (REQUIRED for I/O):
```python
# ✅ Correct - Async for I/O operations
import aiofiles

async def save_data(file_path: str, data: dict) -> None:
    async with aiofiles.open(file_path, 'w') as f:
        await f.write(json.dumps(data))

# ❌ Wrong - Blocking I/O in async context
async def save_data(file_path: str, data: dict) -> None:
    with open(file_path, 'w') as f:  # Blocks event loop!
        json.dump(data, f)
```

**Error Handling**:
```python
# ✅ Correct
from fastapi import HTTPException

try:
    result = await magento_client.get_blocks()
except MagentoAPIError as e:
    raise HTTPException(
        status_code=502,
        detail=f"Magento API error: {str(e)}"
    )
```

**Database Operations**:
```python
# ✅ Correct - Proper transaction management
async with AsyncSessionLocal() as db:
    try:
        # ... operations ...
        await db.commit()
    except Exception as e:
        await db.rollback()  # MUST rollback on error
        raise

# ❌ Wrong - Missing rollback
async with AsyncSessionLocal() as db:
    try:
        # ... operations ...
        await db.commit()
    except Exception as e:
        pass  # Missing rollback!
```

**Avoid N+1 Queries**:
```python
# ✅ Correct - Eager loading
from sqlalchemy.orm import selectinload

query = select(SyncHistory).options(
    selectinload(SyncHistory.source_instance),
    selectinload(SyncHistory.destination_instance)
)
items = await db.execute(query)

# ❌ Wrong - N+1 queries
for item in history_items:
    source = await db.execute(
        select(Instance).where(Instance.id == item.source_instance_id)
    )  # Query in loop!
```

### TypeScript (Frontend)

**No `any` Types**:
```typescript
// ✅ Correct
interface CMSBlock {
  id: number;
  identifier: string;
  content: string;
}

const data: CMSBlock[] = await api.get('/blocks');

// ❌ Wrong
const data: any = await api.get('/blocks');
```

**React Hooks Dependencies**:
```typescript
// ✅ Correct
const loadData = useCallback(async () => {
  const result = await instanceService.getAll();
  setInstances(result);
}, [setInstances]);

useEffect(() => {
  loadData();
}, [loadData]);

// ❌ Wrong - Missing dependencies
useEffect(() => {
  loadData();  // Not in dependencies!
}, []);
```

**Zustand Store Usage**:
```typescript
// ✅ Correct - Use selector
const setInstances = useStore(state => state.setInstances);
setInstances(data);

// ❌ Wrong - Direct store access bypasses React
useStore.getState().setInstances(data);
```

**Performance Optimization**:
```typescript
// ✅ Correct - Memoize expensive computations
const filteredItems = useMemo(() => {
  return items.filter(item =>
    item.status === statusFilter &&
    item.title.includes(searchQuery)
  );
}, [items, statusFilter, searchQuery]);

// ❌ Wrong - Recalculated every render
const filteredItems = items.filter(item =>
  item.status === statusFilter
);
```

---

## Git Flow Workflow

### Branch Structure
```
main (production)           [v1.0.0]──────[v1.1.0]
  ↑                                ↑            ↑
  │                                │            │
  └─── hotfix/v1.0.1 ──────────────┤            │
                                    │            │
                              release/v1.1.0     │
                                    ↑            │
                                    │            │
develop (integration)       ────────┴────────────┴────
  ↑  ↑
  │  └──── feature/new-feature
  └─────── bugfix/fix-error
```

### Critical Rules

**MUST**:
- ✅ Start features with `/feature-start <name>`
- ✅ Review ALL PRs with `/pr-review` before merge
- ✅ Run `/run-tests all` before finishing features
- ✅ Keep hotfixes minimal (production fixes only)
- ✅ Follow branch naming conventions

**MUST NOT**:
- ❌ Commit directly to `main` or `develop`
- ❌ Merge without code review
- ❌ Skip tests before merging
- ❌ Add features to release/hotfix branches
- ❌ Merge features directly to main

### Common Workflows

**Feature Development**:
```bash
/feature-start add-widgets    # Create feature branch
# ... make changes ...
/review                        # Review changes
/run-tests all                # Run tests
/check-types                  # Check types
/feature-finish               # Trigger pr-reviewer, create PR
/pr-review                    # (Reviewer) Review PR
# Merge after approval
```

**Hotfix (Production Emergency)**:
```bash
/hotfix-start critical-bug    # From main
# ... fix ONLY the critical issue ...
/run-tests all
/hotfix-finish               # Creates 2 PRs (main + develop)
# Merge, tag, deploy immediately
```

---

## Development Workflows

### Adding New Features

**Decision Tree**:
```
Is it a new Magento entity type? (e.g., widgets, categories)
├─ YES:
│  1. Add methods to integrations/magento_client.py
│  2. Create service in services/ for comparison/sync
│  3. Add API endpoints in api/
│  4. Create frontend page in pages/
│  5. Add navigation in Layout.tsx
│  6. Update types in types/index.ts and schemas.py
│
└─ NO: Is it a new comparison field?
   ├─ YES:
   │  1. Update services/comparison.py logic
   │  2. Update schemas.py response models
   │  3. Update DiffViewer.tsx display
   │  4. Update types/index.ts
   │
   └─ NO: Is it a UI enhancement?
      1. Update component in components/
      2. Update styles with Material-UI sx prop
      3. Update Zustand store if state needed
```

### Debugging Sync Issues

**Decision Tree**:
```
Sync failed?
├─ Check sync history in UI or database
├─ Review JSON cache: backend/data/instances/{id}/
├─ Test Magento API: /api/instances/{id}/test
├─ Check FastAPI logs for errors
└─ Verify Magento API token permissions
```

**Use Subagents**:
- `/debug-api <instance_id>` - Debug Magento API issues
- `/check-sync` - Validate sync operation safety

---

## Key API Endpoints

### Instance Management
```
GET    /api/instances/                    # List all
POST   /api/instances/                    # Create
PUT    /api/instances/{id}                # Update
DELETE /api/instances/{id}                # Delete
POST   /api/instances/{id}/test           # Test connection
GET    /api/instances/data-snapshots/all  # Get cache status
```

### Comparison
```
POST   /api/compare/blocks                # Compare blocks (uses cache)
POST   /api/compare/pages                 # Compare pages (uses cache)
POST   /api/compare/refresh/{id}          # Refresh cache from Magento
```

### Synchronization
```
POST   /api/sync/preview                  # Preview changes (no mutations)
POST   /api/sync/execute                  # Execute sync (background task)
GET    /api/sync/status/{sync_id}         # Get sync progress
```

### History
```
GET    /api/history/                      # List with filters
GET    /api/history/statistics            # Get stats
GET    /api/history/{sync_id}             # Detailed sync info
```

---

## Specialized Subagents

### When to Use

| Subagent | Trigger | Purpose |
|----------|---------|---------|
| **pr-reviewer** | `/pr-review` | **REQUIRED** before any merge |
| **backend-reviewer** | Auto on .py changes | Review Python/FastAPI code |
| **frontend-reviewer** | Auto on .ts/.tsx | Review React/TypeScript |
| **magento-api-debugger** | `/debug-api <id>` | Debug Magento API issues |
| **sync-validator** | `/check-sync` | Validate sync operations |
| **test-generator** | `/add-test <file>` | Generate comprehensive tests |

### Example: Using pr-reviewer
```bash
# Before merging ANY pull request
/pr-review feature/new-feature develop

# The pr-reviewer checks:
# ✅ Git Flow compliance
# ✅ Code quality (delegates to backend/frontend-reviewer)
# ✅ Tests pass
# ✅ No security issues
# ✅ No merge conflicts
# ✅ Provides APPROVE/REQUEST CHANGES recommendation
```

---

## Security Best Practices

### Input Validation
```python
# ✅ All endpoints use Pydantic validation
class InstanceCreate(BaseModel):
    name: str = Field(..., min_length=1, max_length=100)
    base_url: str = Field(..., regex=r'^https?://')
    api_token: str = Field(..., min_length=32)
```

### SQL Injection Prevention
```python
# ✅ Always use SQLAlchemy ORM (never raw SQL)
query = select(Instance).where(Instance.id == instance_id)

# ❌ Never use raw SQL with user input
query = f"SELECT * FROM instances WHERE id = {instance_id}"  # DANGEROUS!
```

### XSS Prevention
```typescript
// ✅ React escapes by default
<div>{userContent}</div>

// ⚠️ Only use dangerouslySetInnerHTML when absolutely necessary
// and sanitize first
<div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(html) }} />
```

### API Token Storage
```python
# ⚠️ Current: Plain text in database
# TODO: Encrypt tokens at rest using cryptography library
from cryptography.fernet import Fernet

# Before production:
# 1. Generate encryption key
# 2. Encrypt tokens before storage
# 3. Decrypt only when making API calls
```

---

## Common Pitfalls & Solutions

### Pitfall 1: Unnecessary API Calls
```python
# ❌ Wrong
@router.post("/compare/blocks")
async def compare(request: ComparisonRequest):
    # Fetches from Magento API every time!
    source = await magento_client.get_blocks(request.source_id)
    dest = await magento_client.get_blocks(request.dest_id)

# ✅ Correct
@router.post("/compare/blocks")
async def compare(request: ComparisonRequest):
    # Uses cached JSON files
    return await comparison_service.compare_blocks(
        request.source_id,
        request.dest_id
    )
```

### Pitfall 2: Forgetting to Refresh Cache After Sync
```python
# ✅ Correct - Auto-refresh in sync service
async def execute_sync(...):
    # ... perform sync ...
    await db.commit()

    # Refresh source instance cache
    await data_storage.refresh_instance_data(source_instance_id)
```

### Pitfall 3: Blocking UI During Sync
```python
# ✅ Correct - Background task
@router.post("/sync/execute")
async def execute_sync(request: SyncRequest, background_tasks: BackgroundTasks):
    sync_id = create_sync_record(...)

    # Run in background, return immediately
    background_tasks.add_task(
        sync_service.execute_sync,
        sync_id,
        request
    )

    return {"sync_id": sync_id, "status": "started"}
```

---

## Testing Strategy

### Before Committing
```bash
# Backend
cd backend
pytest                          # Run tests
mypy backend/                  # Type checking
black backend/                 # Format code

# Frontend
cd frontend
npm run test                   # Jest tests
npm run type-check             # TypeScript
npm run lint                   # ESLint
npm run build                  # Verify build

# Or use slash commands
/run-tests all
/check-types
```

### Manual Testing Checklist
1. Create instance → Test connection
2. Refresh data → Verify JSON cache
3. Compare blocks/pages → Check diff viewer
4. Preview sync → Verify changeset
5. Execute sync → Check history
6. Verify data refreshed after sync

See `docs/MANUAL_TESTING_CHECKLIST.md` for complete checklist.

---

## Performance Optimization

### Database Queries
```python
# ✅ Use eager loading to avoid N+1
query = select(SyncHistory).options(
    selectinload(SyncHistory.source_instance),
    selectinload(SyncHistory.destination_instance)
)

# ✅ Add indexes on frequently queried fields
class Instance(Base):
    __tablename__ = "instances"

    id = Column(Integer, primary_key=True)
    name = Column(String, index=True)  # Add index
```

### Frontend Performance
```typescript
// ✅ Memoize expensive computations
const filteredItems = useMemo(() => {
  return items.filter(matchesFilter);
}, [items, filter]);

// ✅ Debounce search input
const debouncedSearch = useDebouncedCallback(
  (value) => setSearchQuery(value),
  300
);
```

---

## Quick Reference

### Daily Commands
```bash
# Start development
/feature-start <name>

# During development
/review                 # Review changes
/run-tests all         # Run tests
/check-types           # Type checking

# Finish feature
/feature-finish        # Review + create PR
/pr-review            # (Reviewer) Review PR
```

### Emergency Commands
```bash
# Production is broken!
/hotfix-start <name>
# ... fix issue ...
/run-tests all
/hotfix-finish
```

### Debugging Commands
```bash
/test-api <id>        # Test Magento connection
/debug-api <id>       # Debug API issues
/check-sync           # Validate sync operation
/refresh-data <id>    # Refresh cache
```

---

## Additional Resources

**Comprehensive Documentation**:
- `docs/README.md` - Documentation hub (start here)
- `docs/AI_DEVELOPMENT_GUIDE.md` - Extended AI guide
- `docs/DEVELOPMENT_WORKFLOW.md` - Detailed workflows
- `docs/GIT_FLOW_GUIDE.md` - Complete Git Flow guide
- `docs/CODE_REVIEW_REPORT.md` - Code quality assessment
- `.claude/QUICK_REFERENCE.md` - Command cheat sheet

**External Resources**:
- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [React Documentation](https://react.dev/)
- [Material-UI Components](https://mui.com/)
- [Magento 2 REST API](https://devdocs.magento.com/guides/v2.4/rest/bk-rest.html)

---

## Success Criteria

Before completing any task, verify:

- [ ] Code follows established patterns
- [ ] Type hints complete (Python) / No `any` types (TypeScript)
- [ ] Error handling present
- [ ] Tests written and passing
- [ ] Git Flow compliance maintained
- [ ] Code reviewed (if applicable)
- [ ] Documentation updated
- [ ] No security vulnerabilities introduced

---

**Remember**: When in doubt, consult `docs/` for detailed guidance. Use subagents for specialized tasks. Follow Git Flow strictly. Always review before merging.
