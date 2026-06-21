# Fugue Runtime — Implementation Guide

This document is the single source of truth for implementing the Fugue runtime (`packages/fugue/runtime/`).
It tracks what has been done, what is next, and how to do it.

Read this file at the start of every session before touching any code.
Update the task checklist every time a task is completed.

---

## What we are building

A TypeScript orchestrator at `packages/fugue/runtime/` that executes the full Orchestra skill chain programmatically via the Claude API. When triggered (CLI or GitHub Actions), it runs: breakdown → parallel write-task → parallel verify-task → delegate implement (branch + task doc per task) → reprise. LangSmith traces every LLM call. Token usage is logged per step.

**Repo:** `github.com/codewithbre/orchestra` (local: `C:\code\personal\canon`)

---

## Available skills

These Claude Code skills are available in this session. Use them — do not skip them.

**Planning workflow:**
- `/breakdown` — decompose a feature into executable tasks
- `/write-task` — write a standalone task document
- `/verify-task` — verify task docs are self-contained
- `/create-tracker` — generate this IMPLEMENTATION.md

**Implementation workflow:**
- `/implement` — implement the scoped functional change defined in the task doc
- `/create-pr` — generate the PR description when done
- `/reprise` — post a retrospective GitHub issue after all tasks are merged

---

## How to use this document

**Starting a new session:**
1. Read this file top to bottom.
2. Find the first unchecked task in the checklist below.
3. Read its task document from `.claude/tasks/fugue-runtime/`.
4. Use `/implement` to implement it.
5. Use `/create-pr` to generate the PR body. Open the PR.
6. After merge, check the task off in this file.
7. Move to the next unchecked task.

**If a session ends mid-task:**
- Note the current state in the "Session notes" section at the bottom.
- Do not mark the task complete until the PR is merged.
- Next session: read this file, read the session notes, resume where you left off.

**After all tasks are merged:**
- Run `/reprise` on this file to generate the retrospective GitHub issue.

---

## Task checklist

Tasks must be completed in the order listed. Check a task off only when its PR is merged into main.

---

### Task 01 — Fugue Runtime Project Foundation
**Task doc:** `.claude/tasks/fugue-runtime/task-01-runtime-foundation.md`
**Branch:** `task-01-runtime-foundation`
**What it does:** Creates `packages/fugue/runtime/` with `package.json`, `tsconfig.json`, `.env.example`, and empty src structure. Installs all dependencies.
**Depends on:** Nothing

- [x] Task 01 complete (PR merged) — PR #1, merged 2026-06-20

---

### Task 02 — Skill Loader
**Task doc:** `.claude/tasks/fugue-runtime/task-02-skill-loader.md`
**Branch:** `task-02-skill-loader`
**What it does:** Creates `src/skills.ts` exporting `loadSkill(name: SkillName): string` that reads `skills/*.md` files from the repo root as system prompt strings.
**Depends on:** Task 01

- [x] Task 02 complete (PR merged) — PR #4, merged 2026-06-20

---

### Task 03 — LangChain + LangSmith LLM Client
**Task doc:** `.claude/tasks/fugue-runtime/task-03-llm-client.md`
**Branch:** `task-03-llm-client`
**What it does:** Creates `src/llm.ts` exporting `createLLM()` — a configured `ChatAnthropic` instance with automatic LangSmith tracing when env vars are present.
**Depends on:** Task 01

- [x] Task 03 complete (PR merged) — PR #5, merged 2026-06-20

---

### Task 07 — GitHub Client
**Task doc:** `.claude/tasks/fugue-runtime/task-07-github-client.md`
**Branch:** `task-07-github-client`
**What it does:** Creates `src/github.ts` with `getDefaultBranch`, `createBranch`, `commitFile`, `openPR`, `postComment`, and `createIssue` functions via `@octokit/rest`.
**Depends on:** Task 01

- [x] Task 07 complete (PR merged) — PR #6, merged 2026-06-20

---

### Task 04 — Breakdown Tool
**Task doc:** `.claude/tasks/fugue-runtime/task-04-breakdown-tool.md`
**Branch:** `task-04-breakdown-tool`
**What it does:** Creates `src/tools/breakdown.ts` and `src/types.ts`. `runBreakdown()` calls Claude with the breakdown skill and returns a typed `Task[]` via Zod structured output.
**Depends on:** Tasks 02, 03

- [x] Task 04 complete (PR merged) — PR #8, merged 2026-06-21

---

### Task 05 — Write-Task Tool
**Task doc:** `.claude/tasks/fugue-runtime/task-05-write-task-tool.md`
**Branch:** `task-05-write-task-tool`
**What it does:** Creates `src/tools/write-task.ts`. `runWriteTask()` calls Claude with the write-task skill and returns a `TaskDoc` (`filename` + `content`) ready to commit.
**Depends on:** Tasks 02, 03, 04 (needs `Task` type)

- [x] Task 05 complete (PR merged) — PR #9, merged 2026-06-21

---

### Task 06 — Verify-Task Tool
**Task doc:** `.claude/tasks/fugue-runtime/task-06-verify-task-tool.md`
**Branch:** `task-06-verify-task-tool`
**What it does:** Creates `src/tools/verify-task.ts`. `runVerifyTask()` calls Claude with the verify-task skill and returns `{ confidence, gaps }` via Zod structured output.
**Depends on:** Tasks 02, 03, 05 (needs `TaskDoc` type)

- [x] Task 06 complete (PR merged) — PR #10, merged 2026-06-21

---

### Task 08 — Orchestrator
**Task doc:** `.claude/tasks/fugue-runtime/task-08-orchestrator.md`
**Branch:** `task-08-orchestrator`
**What it does:** Creates `src/orchestrator.ts` with `runFugue()`. Chains all tools end-to-end with parallel fan-out for write-task and verify-task, dependency-ordered implementation, two human-in-the-loop polling gates, and per-step token logging.
**Depends on:** Tasks 04, 05, 06, 07

- [x] Task 08 complete (PR merged) — PR #11, merged 2026-06-21

---

### Task 09 — CLI Entry Point
**Task doc:** `.claude/tasks/fugue-runtime/task-09-cli-entry.md`
**Branch:** `task-09-cli-entry`
**What it does:** Implements `src/index.ts` as a runnable CLI script that parses `--task`, `--repo`, `--issue` args, validates env vars, and calls `runFugue()`.
**Depends on:** Task 08

- [x] Task 09 complete (PR merged) — PR #12, merged 2026-06-21

---

### Task 10 — GitHub Actions Workflow
**Task doc:** `.claude/tasks/fugue-runtime/task-10-github-actions.md`
**Branch:** `task-10-github-actions`
**What it does:** Creates `.github/workflows/fugue.yml` that triggers on issues labeled `fugue` and runs the CLI with the issue title as `--task`.
**Depends on:** Task 09

- [x] Task 10 complete (PR merged) — PR #14, merged 2026-06-21

---

### Task 11 — Reprise Tool
**Task doc:** `.claude/tasks/fugue-runtime/task-11-reprise-tool.md`
**Branch:** `task-11-reprise-tool`
**What it does:** Creates `src/tools/reprise.ts`. `runReprise()` calls Claude with the reprise skill, posts a retrospective GitHub issue labeled `reprise`, and returns the issue URL. Orchestrator calls this as the final step.
**Depends on:** Tasks 02, 03, 07, 08 (updates orchestrator)

- [x] Task 11 complete (PR merged) — PR #13, merged 2026-06-21

---

## Dependency graph

```
Task 01 (foundation)
  ├── Task 02 (skill loader)   ─┐
  ├── Task 03 (LLM client)     ├─ Task 04 (breakdown)  ─┐
  │                             ├─ Task 05 (write-task)  ├─ Task 08 (orchestrator)
  │                             └─ Task 06 (verify-task) ┘       └── Task 09 (CLI)
  └── Task 07 (GitHub client) ──────────────────────────┘                └── Task 10 (Actions)

Task 11 (reprise) depends on 02, 03, 07, 08 — implement alongside Task 08 or after
```

**Safe serial order:** 01 → 02 → 03 → 07 → 04 → 05 → 06 → 08 → 11 → 09 → 10

---

## How to implement a task

Follow these steps exactly. Do not skip any step.

### 1. Read before writing

Open the task document. Read every section: Outcome, Why, Context, Scope, Out of Scope, Acceptance Criteria.

Before writing any code, output:
```
TASK: <task title>
BRANCH: <branch name>
I WILL CREATE/MODIFY:
  - <file> — <what it does>
OUT OF SCOPE (will not touch):
  - <file or area>
ACCEPTANCE CRITERIA:
  1. <criterion>
```

If anything is unclear, stop and ask. Do not guess.

### 2. Create the branch

```bash
git checkout main
git pull origin main
git checkout -b <branch-name>
```

### 3. Implement

Use `/implement` with the task description. Only modify files in the Scope table. Run `pnpm typecheck --filter fugue-runtime` after writing code.

### 4. Verify

Go through every Acceptance Criterion. Do not claim done until all pass.

### 5. Commit and PR

```
feat(<branch-name>): <short description>
```

Use `/create-pr` to generate the PR body.

### 6. After merge

Check the task off above. Run `git checkout main && git pull`. Move to the next task.

---

## File locations reference

| What | Where |
|---|---|
| Task documents | `.claude/tasks/fugue-runtime/` |
| This file | `.claude/tasks/fugue-runtime/IMPLEMENTATION.md` |
| Runtime source | `packages/fugue/runtime/src/` |
| Tool wrappers | `packages/fugue/runtime/src/tools/` |
| Skill definitions | `skills/` (monorepo root) |
| Runtime config | `packages/fugue/runtime/package.json` |
| GitHub Actions | `.github/workflows/fugue.yml` |
| Fugue docs | `packages/fugue/docs/` |

**Key commands:**
```bash
pnpm typecheck --filter fugue-runtime   # type-check the runtime
pnpm install                             # install all workspace deps
node packages/fugue/runtime/src/index.js --task "..." --repo "owner/repo" --issue 1
```

---

## After all tasks are complete

Run `/reprise` to generate the retrospective GitHub issue:
```
/reprise C:\code\personal\canon\.claude\tasks\fugue-runtime\IMPLEMENTATION.md
```

---

## Session notes

```
Last session: 2026-06-20
Current task: Task 01 complete. Task 02 is next.
Status: Task 01 merged (PR #1). Runtime scaffold exists locally at
  packages/fugue/runtime/ — gitignored, not in public repo.
  Task docs and IMPLEMENTATION.md now public on main.
  pnpm install and pnpm --filter fugue-runtime typecheck both pass locally.
Branch: main (clean)
Next action: Task 02 (skill-loader) on branch task-02-skill-loader.
  Tasks 02, 03, and 07 are all unblocked — no mutual dependencies.
Key decisions carried forward:
  - invokeLLM() wrapper (not llm.invoke()) — Task 03
  - fanOut() with allSettled + retry-after — Task 08
  - OrchestrationContext.limits and .failures — Task 08
  - Prompt caching: stable content first in message array — Tasks 04-06
  - pnpm workspace command: pnpm --filter fugue-runtime typecheck (not pnpm typecheck --filter)
```
