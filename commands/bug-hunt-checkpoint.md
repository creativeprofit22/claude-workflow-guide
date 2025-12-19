---
description: Find bugs in tracked files, produce prioritized report (High/Med/Low)
---

# Bug Hunt Checkpoint

Find bugs in the tracked files. Do NOT explore beyond these files.

## Instructions

### 0. Pre-flight Check (Optional)

Check if `.claude/commands/fix.md` exists in the project.

**If it exists:**
1. Run the project's lint/typecheck commands (from the `/fix` command or package.json scripts)
2. If errors are found, inform the user:
```
Pre-flight: Found lint/type errors.

Recommendation: Run /fix first to clear mechanical issues.
This keeps your bug report focused on real logic bugs.

Continue anyway? [Proceeding with bug hunt...]
```
3. Proceed with the bug hunt regardless (this is advisory, not blocking)

**If it doesn't exist:** Skip silently and proceed to Step 1.

### 1. Get Scope

Read CLAUDE.md in the project root. Extract file paths from `Current Focus > Files`.

**If no files listed:** STOP and ask user to update Current Focus with specific file paths:
```
Cannot proceed: No files specified in Current Focus.

Please update CLAUDE.md with the files to analyze:

## Current Focus
Section: [area name]
Files: src/path/to/file1.ts, src/path/to/file2.ts
```

### 2. Read Each File

- Use the Read tool on each file in scope
- Do NOT use Glob, Grep, or Explore agents
- If a file doesn't exist, note it and continue with others

### 3. Analyze for Bugs

Look for these issues in each file:

| Category | What to Find |
|----------|--------------|
| Logic errors | Incorrect conditions, wrong operators, off-by-one |
| Edge cases | Unhandled empty arrays, null inputs, boundary values |
| Null/undefined | Missing null checks, optional chaining gaps |
| Race conditions | Async timing issues, shared state mutations |
| Security | Injection risks, auth gaps, data exposure |
| Error handling | Uncaught exceptions, missing try/catch, silent failures |
| Type safety | Unsafe casts, `any` usage, type mismatches |

### 4. Categorize by Priority

| Priority | Criteria |
|----------|----------|
| **High** | Crashes, data loss, security vulnerabilities, blocking bugs |
| **Medium** | Incorrect behavior, UX bugs, logic that produces wrong results |
| **Low** | Minor edge cases, cosmetic issues, non-blocking quirks |

### 5. Generate Bug Report

Create `reports/bugs-[feature].md` in the project directory.

Use this exact format:

```markdown
# Bug Report: [Feature Name]

Generated: [YYYY-MM-DD]
Feature: [name from Current Focus] (1 of N)

## Scope
Files analyzed:
- [file1]
- [file2]

## High Priority
| # | Location | Description | Impact |
|---|----------|-------------|--------|
| 1 | file:line | What's wrong | Why it matters |

## Medium Priority
| # | Location | Description | Impact |
|---|----------|-------------|--------|

## Low Priority
| # | Location | Description | Impact |
|---|----------|-------------|--------|

## Summary
- High: X bugs
- Medium: Y bugs
- Low: Z bugs
- Total: N bugs
```

**Important:** The Scope section MUST include the file list — this propagates to subsequent pipeline phases.

### 6. Update Pipeline State

Add or update this section in CLAUDE.md (after Current Focus):

```markdown
## Pipeline State
Phase: debugging
Feature: [name from Current Focus Section]
Tier: high
Tier-Status: pending
Reports:
  - bugs: reports/bugs-[feature].md
```

### 7. Update CLAUDE.md & Output Continuation Prompt

**Update CLAUDE.md (KEEP IT LEAN):**
- REPLACE `Last Session` entirely (don't nest "Previous Session" blocks)
- Note how many bugs were found at each priority level
- DELETE any completed items from `Next Steps` (don't strike through)
- DELETE any `Session Log` or history sections if they exist
- Target: CLAUDE.md should stay under 150 lines

**Output this continuation prompt:**

```
## Continuation Prompt

Continue work on [Project Name] at [directory].

**Pipeline Phase**: debugging
**Feature**: [feature name]
**Current Tier**: high - pending

**Scope** (work only on these files):
- [files from the bug report Scope section]

**Reports**:
- bugs: reports/bugs-[feature].md

**Next Action**: Fix high priority bugs from the bug report

**Approach**: Do NOT explore the codebase. Read only the files in Scope above.
```

**STOP here.** Do not continue working. Wait for the user to copy this prompt, clear the context, and paste it to begin the debugging phase.

## Output Summary

When complete, confirm:
```
Bug hunt complete for [feature].

Report: reports/bugs-[feature].md
- High: X bugs
- Medium: Y bugs
- Low: Z bugs

Pipeline state updated. Ready for /debugging-checkpoint.
```

## Guidelines

- Be thorough but precise — every bug needs a specific location
- Don't flag style preferences as bugs
- Include file:line references for every issue
- If no bugs found in a tier, leave the table empty (don't remove it)
- Create the reports/ directory if it doesn't exist
