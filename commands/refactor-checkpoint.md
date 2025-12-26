---
description: Execute refactors from report, hand off to next feature build
---

# Refactor Checkpoint

Execute refactors from the report. Verify no regressions. Hand off to next build.

## Instructions

### 0. Execution Options (Ask First)

Before proceeding, ask the user:

**Question 1 - Parallel Execution:**
> Run with parallel agents? (Execute independent refactors concurrently)
> - Yes - Parallel refactoring
> - No - Sequential execution (default)

**Question 2 - Ultra Think Mode:**
> Enable ultra think mode? (Extended reasoning before each refactor)
> - Yes - Careful analysis before changes
> - No - Standard execution (default)

Store choices and apply throughout this command's execution.

---

### 1. Get Scope & Report

Read CLAUDE.md Pipeline State:

```markdown
## Pipeline State
Phase: refactoring
Feature: [feature name]
Files: [file list]
Refactor-Report: reports/refactors-[feature].md
```

Load the refactor report.

**If no Pipeline State or report:** Ask user for refactor report path.

### 2. Execute Refactors

Work through the report in priority order: High → Medium → Low.

For each refactor:

1. **Read** the file at the specified location
2. **Apply** the suggested fix
3. **Verify** no regressions:
   - Run tests if they exist
   - Check that related code still works
4. **Mark done** in the report (optional: update report file)

**If a refactor can't be applied:**
- Note why
- Skip and continue
- Document in summary

### 3. Quick Validation

After all refactors, run quick checks:

```bash
# Type check
npm run typecheck / tsc --noEmit / mypy / etc.

# Lint
npm run lint / eslint / etc.

# Tests
npm test / pytest / etc.
```

Fix any issues introduced by refactoring.

### 4. Update Pipeline State

```markdown
## Pipeline State
Phase: build
Feature: [next feature]
Features-Remaining: [updated list]
Last-Completed: [feature just finished]
```

### 5. Update CLAUDE.md & Output Continuation

**Update CLAUDE.md (KEEP IT LEAN):**
- REPLACE `Last Session` with refactoring summary
- Update `Pipeline State` to build phase
- Move completed feature to a `## Completed` section if desired

**Output this continuation prompt:**

```
## Continuation Prompt

Continue work on [Project] at [directory].

**Phase**: build
**Next Feature**: [feature name]

**Just Completed**: [previous feature]
- Built, validated, refactored

**Features Remaining**:
- [feature 1]
- [feature 2]
- ...

**Next Action**: Implement [next feature name].

**Key Files**: [relevant files for next feature]

**Approach**: Read CLAUDE.md for feature scope. Implement, then run /build-checkpoint when done.
```

**If ALL FEATURES COMPLETE:**

```
## Continuation Prompt

Continue work on [Project] at [directory].

**All Features Complete**

**Completed**:
- [feature 1] - built, validated, refactored
- [feature 2] - built, validated, refactored
- ...

**Next Action**: Final e2e validation across entire build.
1. Run full test suite
2. Test all API endpoints
3. Verify all UI flows
4. Check cross-feature integration
5. Update documentation

**Approach**: Comprehensive review of the complete implementation.
```

**STOP.** Wait for context clear.

## Output Summary

```
Refactoring complete for: [feature]

Refactors applied:
- High: X/Y
- Medium: X/Y
- Low: X/Y

Validation: [pass/fail]

[Next feature: X / All features complete]

Copy the continuation prompt, clear context, and paste.
```

## Guidelines

- Apply refactors carefully, verify each one
- Don't introduce new features during refactoring
- If tests break, fix before moving on
- Skip refactors that are too risky, document why
- Keep changes focused on the report items
