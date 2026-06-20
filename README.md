# Orchestra

A monorepo of AI coding agents built on a shared philosophy: **constraints enable AI, they don't restrict it.**

---

## Agents

| Package | Role | Status |
|---|---|---|
| [`packages/fugue`](packages/fugue) | Build agent — takes intent, breaks it down, implements as PRs | Active |
| [`packages/coda`](packages/coda) | Test + deploy agent — closes the loop after merge | Planning |

```
Fugue  →  builds (plan → implement → PR)
Coda   →  closes (test → deploy → verify)
```

---

## Shared foundations

**`skills/`** — The seven skill definitions used by all agents. Copy into your project's `.claude/commands/` to get the full workflow:

| Skill | What it does |
|---|---|
| `/breakdown` | Decomposes a feature into scoped, verifiable tasks |
| `/write-task` | Writes a standalone, agent-executable task document |
| `/verify-task` | Confirms each task doc is self-contained and unambiguous |
| `/create-tracker` | Generates an IMPLEMENTATION.md from the task docs |
| `/implement` | Implements a single task within its defined scope |
| `/create-pr` | Generates the PR description on completion |
| `/reprise` | Posts a retrospective GitHub issue: goal vs delivered, shortcomings, skill improvements |

**`templates/`** — Drop-in starting points for adopting this workflow on any project.

**`docs/`** — The philosophy and workflow that all agents are built on.

---

## Philosophy

Three things stay true across every agent in Orchestra:

1. **AI executes. Humans decide.** Task decomposition, architecture, and scope are human responsibilities.
2. **Every unit of work must be verifiable.** A task that cannot be confirmed complete in one pass is not a valid task.
3. **Nothing proceeds without a task document. Nothing merges without a PR.**

See [`docs/philosophy.md`](docs/philosophy.md) for the full argument.

---

## Adopting this workflow on your project

**Step 1 — Install the skills:**
```bash
cp orchestra/skills/*.md your-project/.claude/commands/
```

**Step 2 — Add the CLAUDE.md template:**
```bash
cp orchestra/templates/CLAUDE.md your-project/CLAUDE.md
# Fill in the [PROJECT-SPECIFIC] sections
```

**Step 3 — Start a feature:**
```
/breakdown Build [feature description]
```

---

## Repository structure

```
orchestra/
├── README.md              This file
├── CLAUDE.md              Agent instructions for contributing to Orchestra
├── pnpm-workspace.yaml
├── docs/
│   ├── philosophy.md      Why constraints enable AI
│   ├── workflow.md        The full skill chain with worked example
│   └── architecture.md    Current sequential and future multi-agent models
├── skills/                Six skill definitions — copy into any project
├── templates/             CLAUDE.md and IMPLEMENTATION.md starting points
└── packages/
    ├── fugue/             Build agent
    ├── coda/              Test + deploy agent (planning)
    └── shared/            Common runtime code (planning)
```
