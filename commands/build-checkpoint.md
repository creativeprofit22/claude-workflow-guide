---
description: Implement feature from plan, output continuation for validation
---

# Build Checkpoint

Implement a single feature, then hand off to validation.

## Instructions

### 0. Execution Options (Ask First)

Before proceeding, ask the user:

**Question 1 - Parallel Execution:**
> Run with parallel agents? (Multiple Task agents work concurrently on different parts of the feature)
> - Yes - Spawn parallel agents
> - No - Sequential execution (default)

**Question 2 - Ultra Think Mode:**
> Enable ultra think mode? (Extended reasoning before implementation)
> - Yes - Deep analysis, thorough planning before coding
> - No - Standard execution (default)

Store choices and apply throughout this command's execution.

---

### 1. Get Current Feature

Read CLAUDE.md. Look for:

```markdown
## Pipeline State
Phase: build
Feature: [current feature name]
Features-Remaining: [list or count]
```

**If no Pipeline State or Phase != build:**
- Look for `## Features` or `## Next Steps` to find what to build
- If nothing found, ask user: "What feature should I implement?"

### 2. Get Scope

Extract from CLAUDE.md:
- `Current Focus > Files` — files to work on
- Or infer from feature description

**If scope unclear:** Ask user for key file paths.

### 3. Implement the Feature

Work only on scoped files. For each change:
- Read file first
- Make focused edits
- Don't over-engineer

**Keep it simple.** Implement what's needed, nothing more.

### 4. Update Pipeline State

```markdown
## Pipeline State
Phase: validate
Feature: [feature just implemented]
Files-Changed: [list files you modified/created]
Features-Remaining: [updated list]
```

### 5. Update CLAUDE.md & Output Continuation

**Update CLAUDE.md (KEEP IT LEAN):**
- REPLACE `Last Session` with brief summary of what was implemented
- Update `Pipeline State` as above
- Target: under 150 lines

**Output this continuation prompt:**

```
## Continuation Prompt

Continue work on [Project] at [directory].

**Phase**: validate
**Feature**: [feature name]

**Files to validate** (implementation just completed):
- [file1]
- [file2]

**What was implemented**:
- [1-2 bullets on what was built]

**Next Action**: Run comprehensive validation:
1. Run tests (if exist)
2. Check API endpoints work
3. Verify UI renders correctly
4. Trace wiring (UI → Logic → API)
5. Look for bottlenecks
6. Look for bugs

Fix any issues found, retest until clean.

**Key files**: [entry points, test files, config]
```

**STOP.** Do not validate. Wait for context clear.

## Output Summary

```
Feature implemented: [name]

Files changed:
- [list]

Ready for validation. Copy the continuation prompt, clear context, and paste to begin validation phase.
```

## Guidelines

- One feature at a time
- Don't explore beyond scope
- Don't add unrequested features
- Keep implementation focused
- If blocked, note it and ask user
