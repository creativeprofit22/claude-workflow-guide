---
description: Validate feature (tests, API, UI, wiring, bottlenecks), fix issues, hand off to refactor-hunt
---

# Validate Checkpoint

Comprehensive validation of implemented feature. Fix issues until clean.

## Instructions

### 0. Execution Options (Ask First)

Before proceeding, ask the user:

**Question 1 - Parallel Execution:**
> Run with parallel agents? (Spawn separate agents for: Tests, API, UI, Wiring, Bottlenecks, Bugs)
> - Yes - Parallel validation (faster, uses more resources)
> - No - Sequential checks (default)

**Question 2 - Ultra Think Mode:**
> Enable ultra think mode? (Extended reasoning for each validation check)
> - Yes - Deep analysis, thorough investigation
> - No - Standard execution (default)

Store choices and apply throughout this command's execution.

---

### 1. Get Scope

Read CLAUDE.md Pipeline State:

```markdown
## Pipeline State
Phase: validate
Feature: [feature name]
Files-Changed: [files to validate]
```

**If no Pipeline State:** Ask user what to validate.

### 2. Run Validation Checks

For each file in scope, perform these checks:

#### A. Tests (if exist)
```bash
# Run project test command
npm test / pytest / go test / etc.
```
- Note failures
- If no tests exist, note it

#### B. API Endpoints
- Find API routes in scope
- Test each endpoint:
```bash
curl -X GET/POST http://localhost:PORT/endpoint
```
- Verify responses are correct
- Check error handling

#### C. UI Verification
- Check components render without errors
- Verify props/state flow correctly
- Look for missing error states, loading states
- Check responsive behavior if applicable

#### D. Wiring Check
Trace data flow:
```
UI Component → Hook/State → API Call → Backend → Response → UI Update
```
- Verify imports/exports are correct
- Check data transformations
- Confirm error propagation

#### E. Bottlenecks
Look for:
- N+1 queries
- Unnecessary re-renders
- Missing memoization on expensive operations
- Large bundle imports
- Synchronous operations that should be async

#### F. Bugs
Look for:
- Logic errors
- Edge cases (null, empty, boundary values)
- Race conditions
- Missing error handling
- Type mismatches

### 3. Fix Issues Found

For each issue:
1. Fix it
2. Re-run relevant validation
3. Confirm fixed

**Loop until all checks pass.**

### 4. Generate Validation Report

Create/update `reports/validation-[feature].md`:

```markdown
# Validation Report: [Feature]

Date: [YYYY-MM-DD]

## Files Validated
- [file1]
- [file2]

## Checks Performed

### Tests
- Status: pass/fail/skipped
- Notes: [any failures fixed]

### API Endpoints
| Endpoint | Method | Status | Notes |
|----------|--------|--------|-------|

### UI
- Renders: yes/no
- Issues found: [list]

### Wiring
- Data flow verified: yes/no
- Issues found: [list]

### Bottlenecks
- Found: [count]
- Fixed: [count]
- Remaining: [list if any]

### Bugs
- Found: [count]
- Fixed: [count]

## Summary
- All checks passing: yes/no
- Ready for refactor-hunt: yes/no
```

### 5. Update Pipeline State

```markdown
## Pipeline State
Phase: refactor-hunt
Feature: [feature name]
Files-Validated: [file list]
Validation-Report: reports/validation-[feature].md
```

### 6. Update CLAUDE.md & Output Continuation

**Update CLAUDE.md (KEEP IT LEAN):**
- REPLACE `Last Session` with validation summary
- Update `Pipeline State` as above

**Output this continuation prompt:**

```
## Continuation Prompt

Continue work on [Project] at [directory].

**Phase**: refactor-hunt
**Feature**: [feature name]

**Files to analyze** (just validated):
- [file1]
- [file2]

**Validation status**: Clean (all checks passed)

**Reports**:
- validation: reports/validation-[feature].md

**Next Action**: Find refactoring opportunities in the validated files.

Look for: DRY violations, complexity, naming, dead code, type safety, inconsistent patterns.

**Approach**: Read only the files listed above.
```

**STOP.** Do not refactor-hunt. Wait for context clear.

## Output Summary

```
Validation complete for: [feature]

Checks:
- Tests: [pass/fail/skip]
- API: [pass/fail]
- UI: [pass/fail]
- Wiring: [pass/fail]
- Bottlenecks: [count found/fixed]
- Bugs: [count found/fixed]

Report: reports/validation-[feature].md

Ready for refactor-hunt. Copy the continuation prompt, clear context, and paste.
```

## Guidelines

- Fix issues as you find them, don't just report
- Re-validate after each fix
- Only pass when everything is clean
- If something can't be fixed, document why
- Don't explore beyond scoped files
