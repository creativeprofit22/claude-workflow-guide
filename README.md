# 🚀 Claude Code Checkpoint Workflow Guide

> **Credit:** Massive props to Willem! This guide fuses an existing prompt with his approach to find the optimal hybrid workflow.

📦 **Get the checkpoint command:** [github.com/creativeprofit22/claude-workflow-guide](https://github.com/creativeprofit22/claude-workflow-guide)

---

## 📋 Table of Contents

- [The Ideal Workflow](#-the-ideal-workflow)
- [Fresh Builds: Step by Step](#-fresh-builds-step-by-step)
- [Modifying Builds: Step by Step](#-modifying-builds-step-by-step)

---

## 🔄 The Ideal Workflow

> **Coming Soon:** Three extra commands are planned: `bug-hunt`, `debugging`, and `refactor` to make this workflow even more powerful.

### Complete Development Cycle

```
Build Phase 1
    ↓
🐛 Bug Hunt → 🔧 Debugging → ✨ Refactoring → 🏗️ Build Next Phase
    ↓
Rinse & Repeat
```

### Detailed Steps

#### 1️⃣ Bug Hunt Phase
1. Run `/bug-hunt-checkpoint`
2. Copy the bug-hunt continuation prompt
3. Clear context
4. Paste the continuation prompt
5. **Output:** Bug report

#### 2️⃣ Debugging Phase
1. Run `/debugging-checkpoint`
2. Copy the debugging continuation prompt
3. Clear context
4. Paste the continuation prompt
5. **Output:** Fixes report

#### 3️⃣ Refactoring Phase
1. Run `/refactor-checkpoint`
2. **Output:** Refactoring needs report
3. Copy the refactor continuation prompt
4. Clear context
5. Paste the continuation prompt
6. **Output:** Completion report

#### 4️⃣ Build Next Phase
1. Run the original `/checkpoint` command
2. Copy the continuation prompt
3. Paste the continuation prompt
4. Build the next phase
5. **Repeat the cycle**

> ⚠️ **Important:** Each phase (Bug Hunt, Debugging, Refactoring, Build) should be further divided step-by-step. Run `/checkpoint` between steps as needed. Determine appropriate points to **commit and push**.

---

## 🆕 Fresh Builds: Step by Step

### Prompt #1 — Research Phase

> 💡 **Tip:** Be as specific as possible without writing a book. Break down objectives, features, and architecture ideas into concise paragraphs.

**Template:**
Do some research on GitHub via grep MCP, deploy 5 build-research agents
to determine the highest-quality structure, architecture, coding,
components, and implementation for my project/build.

The structure should be broken down comprehensively and in a modular
manner to ensure the highest quality possible and to facilitate
implementation, exploration, debugging, and testing.

Also mention the GitHub repos and their paths in your report.
```

**After approval:**
- ✅ Commit and push findings to a new local and online repo
- ✅ Ensure your MD mentions the relevant path(s)

---

### Prompt #2 — Execution Planning

> 🧹 Clear context before this step

**Template:**
```
[Point towards your research documentation path]

Draw a step-by-step execution plan, respecting the structure
and architecture of the repo.
```

**After approval:**
- ✅ Push and commit to a new sub-folder in your repo

---

### Prompt #3 — Build Phase

> 🧹 Clear context before this step

**Template:**
```
Deploy [X] number of coding agents to build step/phase 1
```

**After approval:**
1. Run `/checkpoint`
2. Commit and push
3. Copy continuation prompt
4. Clear context
5. Paste in new chat
6. **Continue to next phase**

---

## 🔧 Modifying Builds: Step by Step

### Prompt #1 — Research Improvements

**Template:**
```
Do some research on GitHub via grep MCP, deploy 5 build-research agents
to determine how to improve the structure, architecture, coding,
components, and implementation of the following build:

[Insert the path to your existing build]

The structure should be broken down comprehensively and in a modular
manner to ensure the highest quality possible and to facilitate
implementation, exploration, debugging, and testing.

Also mention the GitHub repos and their paths in your report.
```

**After approval:**
- ✅ Commit and push findings to a new repo
- ✅ Ensure your MD mentions the build you're modifying

---

### Prompt #2 — Execution Planning

> 🧹 Clear context before this step

**Template:**
```
[Point towards your research documentation path]

Draw a step-by-step execution plan, respecting the structure
and architecture of the repo.
```

**After approval:**
- ✅ Push and commit to a new sub-folder

---

### Prompt #3 — Build Phase

> 🧹 Clear context before this step

**Template:**
```
Deploy [X] number of coding agents to build step/phase 1
```

**After approval:**
1. Run `/checkpoint`
2. Commit and push
3. Copy continuation prompt
4. Clear context
5. Paste in new chat
6. **Continue to next phase**

---

## 📝 Quick Reference

| Phase | Command | Output |
|-------|---------|--------|
| Bug Hunt | `/bug-hunt-checkpoint` | Bug report |
| Debugging | `/debugging-checkpoint` | Fixes report |
| Refactoring | `/refactor-checkpoint` | Refactor report |
| Save Progress | `/checkpoint` | Continuation prompt |

---

<div align="center">

**Happy Building!** 🎉

*Remember: Clear context → Paste prompt → Execute → Checkpoint → Repeat*

</div>
