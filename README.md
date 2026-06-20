# Canon

A multi-agent coding workflow system built on a single principle: **constraints enable AI, they don't restrict it.**

Canon gives AI coding agents a structured, verifiable process to follow. The result is that a less capable model with great structure consistently outperforms a smarter model running unconstrained.

---

## Philosophy

Three things stay true across every project canon is applied to:

1. **AI executes. Humans decide.** Task decomposition, architecture, and scope are human responsibilities. Agents implement within defined bounds.
2. **Every unit of work must be verifiable.** A task that cannot be confirmed complete in one pass is not a valid task.
3. **Nothing proceeds without a task document. Nothing merges without a PR.** The structure is the product.

See [`docs/philosophy.md`](docs/philosophy.md) for the full argument.

---

## The Workflow

Canon is a chain of six skills invoked in sequence:

| Step | Skill | What it does |
|---|---|---|
| 1 | `/breakdown` | Decomposes a feature into a list of scoped, verifiable tasks |
| 2 | `/write-task` | Writes a standalone, agent-executable document for each task |
| 3 | `/verify-task` | Confirms each task doc is self-contained and unambiguous |
| 4 | `/create-tracker` | Generates an IMPLEMENTATION.md tracking all tasks and dependencies |
| 5 | `/implement` | Implements a single task within its defined scope |
| 6 | `/create-pr` | Generates the PR description on completion |

See [`docs/workflow.md`](docs/workflow.md) for a worked example tracing a feature through the full chain.

---

## Adopting Canon on a New Project

**Step 1 — Install the skills**

Copy the skill files from [`skills/`](skills/) into your project's `.claude/commands/` directory (or your global `~/.claude/commands/` to use them everywhere):

```bash
cp canon/skills/*.md your-project/.claude/commands/
```

**Step 2 — Add the CLAUDE.md**

Copy [`templates/CLAUDE.md`](templates/CLAUDE.md) into your project root and fill in the `[PROJECT-SPECIFIC]` sections.

**Step 3 — Start a feature**

When you have a feature to build, give it to your agent with:

```
/breakdown Build [feature description]
```

The agent will decompose it, write task docs, verify them, generate a tracker, and implement each task as a PR. You review and merge each PR before the next task begins.

---

## What canon is not

- It is not a framework that auto-merges or deploys without you.
- It is not a replacement for architectural judgment — you still decide what to build.
- It is not opinionated about language, framework, or toolchain — it works on any codebase.

---

## Repository structure

```
canon/
├── README.md              This file
├── CLAUDE.md              Agent instructions for contributing to canon itself
├── docs/
│   ├── philosophy.md      Why constraints enable AI
│   ├── workflow.md        The full skill chain with worked example
│   └── architecture.md    Current and future multi-agent architecture
├── skills/                The six skill definitions (copy these into your project)
│   ├── breakdown.md
│   ├── write-task.md
│   ├── verify-task.md
│   ├── create-tracker.md
│   ├── implement.md
│   └── create-pr.md
└── templates/
    ├── CLAUDE.md          Template CLAUDE.md for adopting projects
    └── IMPLEMENTATION.md  Template tracker for adopting projects
```
