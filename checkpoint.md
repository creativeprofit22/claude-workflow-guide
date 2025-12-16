---
description: Save progress to CLAUDE.md AND generate continuation prompt
---

# Session Checkpoint Command

Save current session state to CLAUDE.md for long-term persistence AND output a continuation prompt for immediate use.

## Instructions

Do TWO things:

### 1. Update CLAUDE.md

Read the existing CLAUDE.md in the project root. Update these sections based on the current session:

- **Last Session** - Update with today's date and what was done
- **Current Focus** - Update if it changed
- **Next Steps** - Update based on where we're leaving off

Preserve everything else in the file. Write the changes.

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

- Don't bloat CLAUDE.md — keep updates concise
- Don't bloat the continuation prompt — CLAUDE.md has the details
- Always include the project path in the continuation prompt
- Be specific with file paths and function names
- Prioritize actionable next steps
