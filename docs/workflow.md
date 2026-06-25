# Workflow

The Orchestra workflow is an ordered pipeline of skills. Each phase has a
defined input, a defined output, and a human checkpoint before the next phase
begins. Phases run in dependency order; within a phase, independent work can
run in parallel when the task breakdown allows it. Run it interactively in
Claude Code or programmatically via the Octave runtime.

```
/workflow → /breakdown → /write-task → /verify-task → /create-tracker
         → /implement → [Human UAT] → /feedback → /create-pr → /reprise
```

---

## The steps

### 1. /workflow

**Input:** A description of the work to be done.
**Output:** A completed planning and implementation cycle.
**Human checkpoint:** The breakdown is the only gate where the plan can change.

Orchestrates the full cycle. Runs breakdown, enforces the human approval gate,
then runs write-task, verify-task, and create-tracker in phase order. Write-task
and verify-task fan out across tasks when run programmatically; implement runs
in parallel for tasks with no outstanding dependencies. After implementation
and UAT, routes to feedback or create-pr.

---

### 2. /breakdown

**Input:** A feature description or problem statement.
**Output:** An ordered, confidence-rated task list with dependencies, plus an
`overview.md` capturing the codebase analysis done during breakdown.
**Human checkpoint:** Review and approve the task list. Reject or refine any
task that is too vague, too large, or has unclear acceptance criteria.

The breakdown is where scope, intent, and task boundaries are decided. Every
step after formalizes and verifies that decision. The quality of the breakdown
determines the quality of everything that follows.

Each task receives a confidence rating (HIGH / MEDIUM / LOW). LOW confidence
tasks surface a gap that needs human direction before a task document can be
written.

---

### 3. /write-task

**Input:** One approved task from the breakdown.
**Output:** A standalone markdown document in `.claude/tasks/` that contains
everything an isolated agent needs to implement the task from scratch.
**Human checkpoint:** Read the document before verification. If anything is
wrong, fix it now, not during implementation.

The task document is the delegation contract. The agent implementing it will
have access only to the document and the codebase, no conversation history,
no other task documents, no shared context. All relevant context must be in
the file.

Invoke once per task. Each invocation produces exactly one document.

---

### 4. /verify-task

**Input:** A task document or a directory of task documents.
**Output:** A confidence rating per document and a gap report, anything that
would prevent a cold agent from completing the task correctly.
**Human checkpoint:** Act on gaps before proceeding. REWRITE findings must be
addressed. ADD CONTEXT findings should be addressed. ASK AUTHOR findings
require your input.

Run on the full task directory to verify all documents in one pass. The
verifier agent reads each document cold, exactly as the implementing agent
would.

---

### 5. /create-tracker

**Input:** A directory of verified task documents.
**Output:** An IMPLEMENTATION.md with the task checklist, dependency graph,
and session handoff protocol.
**Human checkpoint:** Review the dependency order. Correct it if the agent
misread dependencies.

The tracker is the session handoff document. Any agent in any future session
can read it and know exactly where work stands.

---

### 6. /implement

**Input:** A task document.
**Output:** Code committed to a branch, verified against each acceptance
criterion, ready for PR.
**Human checkpoint:** Review the PR before merging.

The implementing agent reads the task document in full, restates what it will
change and what is out of scope, implements only within those bounds, runs the
project's checks, and verifies each acceptance criterion before reporting
complete.

Designed for a low-reasoning model. By the time a task reaches this step,
there is no ambiguity left to resolve.

---

### 7. /feedback

**Input:** A UAT finding, what was observed versus what was expected.
**Output:** A correction appended to the existing task document without
modifying its original content.
**Human checkpoint:** Confirm the gap is accurately described before the
correction is written.

Used when UAT reveals the outcome fell short of the stated intention. The
correction is implemented before /create-pr is invoked.

---

### 8. /create-pr

**Input:** The current branch diff.
**Output:** A PR summary proving the intention was met: intention, outcome,
value, changes, verification steps.
**Human checkpoint:** Review the PR on GitHub. Merge when satisfied.

---

### 9. /reprise

**Input:** The IMPLEMENTATION.md for the completed project.
**Output:** A GitHub issue labeled `reprise` containing: original goal, what
was delivered, what worked, shortcomings by step, suggested improvements, and
metrics.
**Human checkpoint:** Read the issue. Use findings to improve the skill
definitions for the next project.

Closes the feedback loop. Without it, shortcomings surface only in conversation
and are forgotten. With it, they become a permanent record that improves the
system over time.

---

## Programmatic execution

The full pipeline can also be run by the Octave runtime without manual skill
invocations. Octave takes an intent from a GitHub issue, runs breakdown,
fans out write-task and verify-task in parallel, commits task documents as a
PR, and waits at two human gates: breakdown approval (Gate 1) and PR merge
(Gate 2). Implementation fans out by dependency group: tasks with no blockers
run concurrently; dependent tasks wait until their prerequisites merge.

See `packages/octave/docs/runtime-design.md` for the full runtime design.

---

## Re-entering the workflow mid-chain

- **Adding a task to an existing project:** Start at `/write-task`, skip `/breakdown`.
- **Resuming after a gap:** Read IMPLEMENTATION.md, find the first unchecked task, go straight to `/implement`.
- **UAT revealed a gap:** Use `/feedback` to amend the task document. Do not modify the original content.
- **Scope creep during implementation:** Stop. Write a new task doc with `/write-task`. The current task's scope does not change.
