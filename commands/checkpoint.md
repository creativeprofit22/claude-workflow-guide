---
description: Save progress to CLAUDE.md AND generate continuation prompt
---

# Session Checkpoint Command

Save current session state to CLAUDE.md for long-term persistence AND output a continuation prompt for immediate use.

## Instructions

Do TWO things:

### 0. Check for Pipeline State (Do This First)

Read CLAUDE.md. If a `## Pipeline State` section exists, this is a **pipeline-aware checkpoint**.

Extract:
- **Phase**: build, validate, refactor-hunt, or refactoring
- **Feature**: The feature being worked on
- **Files**: Files in current scope
- **Reports**: Paths to validation and/or refactor reports

If Pipeline State exists, generate a **Pipeline-Aware Continuation Prompt** (see below).

### 1. Update CLAUDE.md (KEEP IT LEAN)

Read the existing CLAUDE.md in the project root. Apply these rules strictly:

**Last Session** - REPLACE entirely (do NOT nest "Previous Session" blocks):
- Delete the old Last Session content completely
- Write only the current session's summary
- Keep it to 5-10 lines max

**Next Steps** - REMOVE completed items (do NOT use strikethrough):
- Delete any items that are done
- Keep only pending/future items
- Renumber the list

**Current Focus** - Update if it changed

**Session Log / History sections** - DELETE if they exist:
- Remove any `## Session Log` section
- Remove any accumulated history
- That's what git is for

**Preserve only**: Project description, Current Focus, Pipeline State, Last Session, Next Steps

Write the changes.

If CLAUDE.md doesn't exist, create it using this structure:

```markdown
# Project Name

One-line description.

## Current Focus
Section: [current area of work]
Files: [relevant files]

## Last Session ([date])
- What was done
- Stopped at: [where you left off]

## Next Steps
1. [First thing to do]
2. [Second thing]
3. [Third thing]
```

### 2. Output Continuation Prompt

After updating the file, output a ready-to-use continuation prompt for immediate context clearing:

```
## Continuation Prompt

Continue work on [Project Name] at [directory].

**What's Done**: [1-2 bullets max]

**Current State**: [where we left off - be specific]

**Next Step**: [the very next thing to do]

**Key Files**: [2-3 file paths relevant to next step]

**Key Context**: [anything critical to remember]

**Approach**: Do NOT explore the full codebase. Use the context above.
```

Keep the continuation prompt SHORT (under 15 lines). The detailed state is now in CLAUDE.md — the prompt just needs enough to bridge the context clear.

### Pipeline-Aware Continuation Prompts

If Pipeline State was detected in step 0, use these formats:

**For build phase:**
```
## Continuation Prompt

Continue work on [Project Name] at [directory].

**Phase**: build
**Feature**: [feature name]

**Files**:
- [scoped files]

**What's Done**: [brief summary]

**Next Action**: Continue implementing [feature], then run /build-checkpoint when done.

**Approach**: Read only the files listed above.
```

**For validate phase:**
```
## Continuation Prompt

Continue work on [Project Name] at [directory].

**Phase**: validate
**Feature**: [feature name]

**Files to validate**:
- [files from Pipeline State]

**Next Action**: Run comprehensive validation:
1. Run tests
2. Check API endpoints
3. Verify UI renders
4. Trace wiring
5. Look for bottlenecks/bugs

Fix issues, retest until clean. Then run /validate-checkpoint.

**Approach**: Read only the files listed above.
```

**For refactor-hunt phase:**
```
## Continuation Prompt

Continue work on [Project Name] at [directory].

**Phase**: refactor-hunt
**Feature**: [feature name]

**Files to analyze**:
- [files from Pipeline State]

**Reports**:
- validation: [path if exists]

**Next Action**: Run /refactor-hunt-checkpoint to find refactoring opportunities.

**Approach**: Read only the files listed above.
```

**For refactoring phase:**
```
## Continuation Prompt

Continue work on [Project Name] at [directory].

**Phase**: refactoring
**Feature**: [feature name]

**Files**:
- [files from Pipeline State]

**Reports**:
- refactors: [path]

**Next Action**: Execute refactors from report, then run /refactor-checkpoint.

**Approach**: Read only the files listed above.
```

## Workflow

User runs `/checkpoint`. Two things happen:

1. CLAUDE.md is updated (long-term memory saved)
2. Continuation prompt is displayed (short-term handoff ready)

User can then:
- **Clear now**: Copy prompt → `/clear` → paste → keep working
- **Stop for the day**: Just close. CLAUDE.md has everything for next time.
- **Keep working**: Do nothing, state is saved anyway

## Adapt to Session Complexity

**Simple session** (quick fix, single task):
- Brief CLAUDE.md update (1-2 bullet points)
- Minimal continuation prompt (5-8 lines)

**Standard session** (feature work, multiple tasks):
- Full CLAUDE.md update
- Standard continuation prompt with all sections

**Complex session** (architecture changes, many files):
- Thorough CLAUDE.md update with decisions documented
- Detailed continuation prompt including key decisions and gotchas

## Guidelines

**CLAUDE.md should stay under 150 lines.** If it's longer, you're doing it wrong.

- REPLACE Last Session, don't append to it
- DELETE completed Next Steps, don't strike them through
- DELETE Session Log / History sections entirely
- Keep the continuation prompt SHORT (under 15 lines)
- Be specific with file paths and function names
- Detailed history belongs in git, not CLAUDE.md
