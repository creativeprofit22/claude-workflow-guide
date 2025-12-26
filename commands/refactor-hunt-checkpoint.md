---
description: Find refactoring opportunities, produce report, hand off to refactoring
---

# Refactor Hunt Checkpoint

Find refactoring opportunities in validated files. Produce prioritized report.

## Instructions

### 1. Get Scope

Read CLAUDE.md Pipeline State:

```markdown
## Pipeline State
Phase: refactor-hunt
Feature: [feature name]
Files-Validated: [files to analyze]
```

**If no Pipeline State:** Ask user for files to analyze.

### 2. Read Each File

- Use Read tool on each file in scope
- Do NOT explore beyond these files

### 3. Analyze for Refactors

Look for:

| Category | What to Find |
|----------|--------------|
| DRY violations | Duplicated code, copy-paste patterns |
| Complexity | Long functions, deep nesting, high cyclomatic complexity |
| Naming | Unclear variable/function names, misleading names |
| Abstractions | Missing interfaces, god objects, feature envy |
| Patterns | Inconsistent coding patterns within the feature |
| Dead code | Unused variables, unreachable code, commented-out code |
| Type safety | Weak typing, excessive `any`, missing generics |

### 4. Categorize by Priority

| Priority | Criteria |
|----------|----------|
| **High** | Tech debt blocking progress, DRY violations, high complexity |
| **Medium** | Code clarity, maintainability, inconsistent patterns |
| **Low** | Style consistency, minor improvements, nice-to-have |

### 5. Estimate Effort

- **S (Small)**: Quick fix, single location
- **M (Medium)**: Some thought needed, few locations
- **L (Large)**: Significant change, multiple files

### 6. Generate Refactor Report

Create `reports/refactors-[feature].md`:

```markdown
# Refactor Report: [Feature]

Generated: [YYYY-MM-DD]

## Scope
Files analyzed:
- [file1]
- [file2]

## High Priority
| # | Location | Issue | Suggested Fix | Effort |
|---|----------|-------|---------------|--------|
| 1 | file:line | Problem | Solution | S/M/L |

## Medium Priority
| # | Location | Issue | Suggested Fix | Effort |
|---|----------|-------|---------------|--------|

## Low Priority
| # | Location | Issue | Suggested Fix | Effort |
|---|----------|-------|---------------|--------|

## Summary
- High: X refactors
- Medium: Y refactors
- Low: Z refactors
- Total: N refactors
```

### 7. Handle Empty Results

**If no refactors found:**
- Skip refactoring phase
- Output continuation for next feature build instead

### 8. Update Pipeline State

```markdown
## Pipeline State
Phase: refactoring
Feature: [feature name]
Files: [file list]
Refactor-Report: reports/refactors-[feature].md
Refactors-Remaining: [total count]
```

### 9. Update CLAUDE.md & Output Continuation

**Update CLAUDE.md (KEEP IT LEAN):**
- REPLACE `Last Session` with refactor-hunt summary
- Update `Pipeline State` as above

**If refactors found, output:**

```
## Continuation Prompt

Continue work on [Project] at [directory].

**Phase**: refactoring
**Feature**: [feature name]

**Files to refactor**:
- [file1]
- [file2]

**Refactor Report**: reports/refactors-[feature].md
- High: X
- Medium: Y
- Low: Z

**Next Action**: Execute refactors from report. Start with High priority.

For each refactor:
1. Read the location
2. Apply the fix
3. Verify no regressions

**Approach**: Read only the files listed above.
```

**If NO refactors found, output:**

```
## Continuation Prompt

Continue work on [Project] at [directory].

**Phase**: build
**Feature**: [next feature name]

**Previous feature complete**: [feature name]
- Built, validated, no refactors needed

**Next Action**: Implement the next feature.

**Features remaining**:
- [list from CLAUDE.md]

**Approach**: Read CLAUDE.md for feature details and scope.
```

**STOP.** Wait for context clear.

## Output Summary

```
Refactor hunt complete for: [feature]

Report: reports/refactors-[feature].md
- High: X
- Medium: Y
- Low: Z
- Total: N

[Ready for refactoring / No refactors needed - ready for next feature]

Copy the continuation prompt, clear context, and paste.
```

## Guidelines

- Focus on actionable refactors, not theoretical improvements
- Don't flag "just different" — only actual issues
- Include specific file:line references
- Be realistic with effort estimates
- If a file is clean, say so
