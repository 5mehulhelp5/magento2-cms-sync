---
description: Generate tests for a specific file or component
argument-hint: <file_path>
allowed-tools: Read, Write, Task, Glob, Grep
---

Generate comprehensive tests for: $ARGUMENTS

Process:
1. Read the target file to understand its functionality
2. Identify all functions/methods/components to test
3. Use the test-generator subagent to create appropriate tests
4. Create test file in the correct location:
   - Backend: `backend/tests/test_<module_name>.py`
   - Frontend: `frontend/src/<path>/__tests__/<ComponentName>.test.tsx`
5. Include fixtures, mocks, and edge cases
6. Verify imports and dependencies

If no file path is provided, ask which file should be tested.
