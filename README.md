# Claude Code Workflow Guide

Efficient context management for multi-project work with Claude Code.

---

## Core Concept

`CLAUDE.md` in your project root = your project's memory. Claude reads it automatically at chat start.

---

## Starting a New Chat

```
Let's work on /path/to/your/project
```

That's it. Claude navigates there, reads `CLAUDE.md`, knows everything.

---

## Ending a Session

```
Update CLAUDE.md with today's progress and give me the project path
```

You get:
- Updated docs for next session
- Path to paste in next chat

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

| Action | What to Say |
|--------|-------------|
| Start session | `Let's work on [path]` |
| Focus on section | `Focus on [files/folders]` |
| End session | `Update CLAUDE.md with progress and give me the path` |
| Check project | `What project is this?` |
| Avoid breaks | `[X] connects to [Y] - don't break that` |

---

## Why This Works

- **No repeated explanations** - docs hold context
- **Minimal token usage** - read only what's needed
- **Clean handoffs** - next chat picks up instantly
- **Nothing breaks** - interfaces documented

---

*Keep it simple. Let the docs do the talking.*
