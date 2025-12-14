# Claude Code Workflow Guide

Efficient context management for multi-project work with Claude Code.

---

## Core Concept

`CLAUDE.md` in your project root = your project's memory. Claude reads it automatically at chat start.

---

## The `/checkpoint` Command

One command that handles everything — saves to file AND gives you a continuation prompt.

### Setup

Create `~/.claude/commands/checkpoint.md` with [this content](checkpoint.md).

### What It Does

When you run `/checkpoint`:

1. **Updates CLAUDE.md** — Progress saved to disk (survives closing terminal)
2. **Outputs a continuation prompt** — Ready to copy if you need to clear context

### When To Use It

| Situation | What Happens |
|-----------|--------------|
| Context getting full, keep working | Copy prompt → `/clear` → paste → continue |
| Done for the day | Just close. CLAUDE.md is saved for tomorrow |
| Quick save mid-session | State captured either way |

---

## Starting a New Chat

```
Let's work on /path/to/your/project
```

That's it. Claude navigates there, reads `CLAUDE.md`, knows everything.

---

## Mid-Session: Context Getting Full

Run `/context` to check usage. When it's getting high:

```
/checkpoint
```

Then either:
- **Keep working**: Copy the prompt → `/clear` → paste → continue with fresh context
- **Take a break**: Just leave it. CLAUDE.md is already updated.

---

## Ending a Session

```
/checkpoint
```

Close the terminal. Tomorrow, start with "Let's work on /path/to/project" and you're back.

---

## CLAUDE.md Template

```markdown
# Project Name

One-line description.

## Current Focus
Section: [which part you're working on]
Files: [specific files]

## Last Session ([date])
- What was done
- Stopped at: [where you left off]

## Next Steps
1. First thing to do
2. Second thing
3. Third thing

## Sections
- **Module A:** docs/a.md, src/a/
- **Module B:** docs/b.md, src/b/

## Interfaces (don't break)
- Module A connects to B via [X]

## Don't Touch
- [Stable modules not being worked on]
```

---

## Working on Specific Sections

Instead of loading entire project context:

```
Let's work on the auth module. Focus on docs/auth.md and src/auth/
```

Claude only reads those files. Minimal tokens.

---

## Multi-Project Setup

Keep a note with your active projects:

```
Project A: /home/user/projects/project-a
Project B: /home/user/projects/project-b
```

Paste the right path when starting a chat.

---

## Project Structure for Efficiency

```
/your-project
├── CLAUDE.md           # High-level index
├── docs/
│   ├── module-a.md     # Deep dive per section
│   ├── module-b.md
│   └── module-c.md
├── src/
│   ├── module-a/
│   ├── module-b/
│   └── module-c/
```

- `CLAUDE.md` = overview + current state
- `docs/*.md` = detailed section docs
- Tell Claude which section to focus on

---

## Quick Reference

| Action | What to Do |
|--------|------------|
| Start session | `Let's work on [path]` |
| Check context | `/context` |
| Save progress + get continuation prompt | `/checkpoint` |
| Clear and continue | Copy prompt → `/clear` → paste |
| Focus on section | `Focus on [files/folders]` |

---

## Why This Works

- **No repeated explanations** — docs hold context
- **Minimal token usage** — read only what's needed
- **Clean handoffs** — next chat picks up instantly
- **Nothing breaks** — interfaces documented
- **Mid-session covered** — `/checkpoint` handles context bloat

---

*One command. Both problems solved.*
