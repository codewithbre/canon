# Workflow

The canon workflow is a chain of six skills invoked in sequence. Each step has a defined input, a defined output, and a human checkpoint before the next step begins.

```
/breakdown → /write-task → /verify-task → /create-tracker → /implement → /create-pr
```

---

## The six steps

### 1. /breakdown

**Input:** A feature description or problem statement from the human.
**Output:** An ordered list of scoped, verifiable tasks with dependencies.
**Human checkpoint:** Review and approve the task list before moving to step 2.

The breakdown agent reads the codebase, identifies what needs to change, and produces tasks that each meet the Task Integrity criteria: single functional change, clear intent, clear location, verifiable in one pass.

Reject tasks that are too broad. Ask the agent to re-scope them. The quality of the breakdown determines the quality of everything that follows.

---

### 2. /write-task

**Input:** One approved task from the breakdown.
**Output:** A standalone markdown document in `.claude/tasks/` that contains everything an isolated agent needs to implement the task from scratch.
**Human checkpoint:** Read the doc before verification. If anything seems wrong, fix it now — not during implementation.

The task doc must be self-contained. The agent implementing it will have access only to the codebase and the document — no conversation history, no other task docs, no shared context.

Invoke `/write-task` once per task. For 8 tasks, invoke it 8 times.

---

### 3. /verify-task

**Input:** A task document or a directory of task documents.
**Output:** A gap report — confidence rating per document, list of anything that would block a cold agent from completing it.
**Human checkpoint:** Act on gaps before proceeding. A REWRITE finding must be addressed; ADD CONTEXT findings should be addressed; ASK AUTHOR findings require your input.

Run `/verify-task` on the full tasks directory at once to verify all docs in one pass.

---

### 4. /create-tracker

**Input:** A directory of verified task documents.
**Output:** An IMPLEMENTATION.md file with the task checklist, dependency graph, implementation protocol, PR template, and session notes section.
**Human checkpoint:** Review the dependency order. Correct it if the agent misread dependencies.

The tracker is the session handoff document. Any agent in any future session can read it and know exactly where things stand.

---

### 5. /implement

**Input:** A task document (specified by the human or read from IMPLEMENTATION.md).
**Output:** Code changes committed to a branch, ready for PR.
**Human checkpoint:** Review the PR before merging. Check the task off in IMPLEMENTATION.md after merge.

The implement agent reads the task doc in full, restates what it will change and what is out of scope, then executes only within those bounds. One task per session.

---

### 6. /create-pr

**Input:** The current branch diff against main.
**Output:** A formatted PR title, summary, file list, verification checklist, and scope confirmation.
**Human checkpoint:** Review the PR on GitHub. Merge when satisfied.

---

## Worked example

**Feature request:** "Add a `getIntervalSemitones` function to the theory package."

**Step 1 - /breakdown:**
Agent reads `packages/theory/src/intervals.ts`, `src/index.ts`, and the existing test structure. Produces one task:
> Add `getIntervalSemitones(interval: IntervalId): number` to `intervals.ts`, export from `index.ts`.

Human approves.

**Step 2 - /write-task:**
Agent writes `.claude/tasks/task-01-get-interval-semitones.md` containing: outcome (what the function returns), why (direct lookup, root/mode do not affect semitone count), context (relevant files and types), scope (two files), out of scope (no changes to engine.ts or scales.ts), acceptance criteria (five verifiable conditions).

Human reads it. Looks good.

**Step 3 - /verify-task:**
Agent reads the doc cold. Reports: confidence HIGH, no gaps found.

**Step 4 - /create-tracker:**
Agent reads the single task doc, generates IMPLEMENTATION.md with checklist (one task), dependency graph (none), and protocol.

**Step 5 - /implement:**
Agent checks out branch `task-01-get-interval-semitones`, adds the function, exports it, runs typecheck. Opens PR.

**Step 6 - /create-pr:**
Agent produces: `feat(theory): add getIntervalSemitones lookup function` with summary, file list (`intervals.ts`, `index.ts`), and verification steps. Human reviews, merges.

Total time on task: one session. Total human decisions: approve breakdown, read task doc, review PR.

---

## Re-entering the workflow mid-chain

You don't always start at step 1.

- **Adding a task to an existing project:** Start at `/write-task`, skip `/breakdown`.
- **Resuming after a gap:** Read IMPLEMENTATION.md, find the first unchecked task, go straight to `/implement`.
- **Something broke in a completed task:** Write a new task doc with `/write-task` for the fix. Do not amend the original.
- **Scope creep during implementation:** Stop. Write a new task for the additional work. The current task's scope does not change.
