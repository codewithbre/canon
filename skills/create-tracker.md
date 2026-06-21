Create an IMPLEMENTATION.md tracker document from a set of completed task documents.
This is the 4th and final step in the planning workflow:
/breakdown → /write-task → /verify-task → /create-tracker

Do not implement anything. This is documentation only.

$ARGUMENTS should be the path to the directory containing task documents (e.g. `.claude/tasks/`).
If no path is provided, look for task documents in the current project's `.claude/tasks/` directory.
If still unclear, ask.

1. SCAN the target directory and read every task document found.
   Skip README files, index files, and non-task files.
   Task documents are identified by having the standard sections: Outcome, Why, Context, Scope, Out of Scope, Acceptance Criteria.

2. EXTRACT from each task document:
   - Task title (from the # heading)
   - One-line summary of what the task does (from Outcome, condensed)
   - Branch name (from the task doc if stated, otherwise derive from the filename: task-01-name.md → task-01-name)
   - Dependencies (from "Depends on" or "Dependency" sections)

3. PRODUCE an IMPLEMENTATION.md with exactly these sections:

---

# <Project Name> — Implementation Guide

This document is the single source of truth for implementing <project name>.
It tracks what has been done, what is next, and how to do it.

Read this file at the start of every session before touching any code.
Update the task checklist every time a task is completed.

---

## How to use this document

**Starting a new session:**
1. Read this file top to bottom.
2. Find the first unchecked task in the checklist below.
3. Read its task document from `.claude/tasks/`.
4. Implement it. Create the PR. Merge it.
5. Check it off in this file.
6. Move to the next unchecked task.

**If a session ends mid-task:**
- Note the current state in the "Session notes" section at the bottom.
- Do not mark the task complete until the PR is merged.
- Next session: read this file, read the session notes, resume where you left off.

---

## Task checklist

Tasks must be completed in the order listed. Check a task off only when its PR is merged into main.

[For each task, produce a block like:]

### <Task title>
**Task doc:** `.claude/tasks/<filename>`
**Branch:** `<branch-name>`
**What it does:** <one-line summary>
**Depends on:** <dependencies, or "Nothing">

- [ ] <Task title> complete (PR merged)

---

## Dependency graph

[Produce an ASCII dependency graph showing which tasks depend on which.
Format:
Task A
  └── Task B (depends on A)
        └── Task C (depends on B)
  └── Task D (depends on A, parallel with B)
]

**Safe serial order:** <list task titles in the safe sequential order>

---

## Pre-flight checklist

Before the first implementation session, verify the following are either
already present or covered by a task doc:

- [ ] Workspace config (`pnpm-workspace.yaml`, `turbo.json`, etc.) recognizes
      the new package. If not, add it to the first task's scope.
- [ ] Runtime devDependency for executing TypeScript exists (`tsx`, `ts-node`,
      or equivalent). If not, add it to the first task's `package.json`.
- [ ] Cross-platform run command is documented in Session notes.
      Use `pnpm --filter <name> <script>` not `pnpm <script> --filter <name>`.
- [ ] If any task creates an LLM client or tool wrapper, a model selection
      table exists in that task's Context section.
- [ ] If programmatic orchestration will invoke write-task or reprise via API,
      `-api` skill variants (`write-task-api.md`, `reprise-api.md`) exist.

---

## How to implement a task

Follow these steps exactly for every task. Do not skip any step.

### 1. Read before writing

Open the task document listed in the checklist above.
Read every section: Outcome, Why, Context, Scope, Out of Scope, Acceptance Criteria.

Before writing any code, output this:

```
TASK: <task title>
BRANCH: <branch name>
I WILL CREATE/MODIFY:
  - <file 1> — <what it does>
  - <file 2> — <what it does>
OUT OF SCOPE (will not touch):
  - <file or area>
ACCEPTANCE CRITERIA:
  1. <criterion 1>
  2. <criterion 2>
```

If anything in the task doc is unclear, stop and ask. Do not guess.

### 2. Create the branch

```bash
git checkout main
git pull origin main
git checkout -b <branch-name>
```

### 3. Implement

- Only create or modify files listed in the task doc's Scope table.
- Do not touch files listed in Out of Scope.
- Run the project's typecheck command after writing code.
- Run the project's formatter before committing.

### 4. Verify acceptance criteria

Go through the task doc's Acceptance Criteria one by one.
Do not claim a task is done until every criterion is met.

### 5. Commit

One commit per logical unit of work within the task.
Format: `feat(<branch-name>): <short description>`

### 6. Open the PR

**PR title:** matches the task doc title exactly.

**PR body:**
```
## What this does
<Restate the Outcome section from the task doc in 1-2 sentences.>

## Why
<Restate the Why section from the task doc in 1 sentence.>

## Changes
| File | Action |
|---|---|
| `path/to/file` | Created / Modified |

## Verification
- [ ] <Acceptance criterion 1>
- [ ] <Acceptance criterion 2>
- [ ] No files outside Scope were modified
```

### 7. After PR is merged

Check off the task in the Task checklist above.
Pull main locally: `git checkout main && git pull origin main`.
Move to the next unchecked task.

---

## File locations reference

[List the key file locations specific to this project — task docs directory, source directories, config files, etc.]

---

## Session notes

Use this section to record state when ending a session mid-task.

```
Last session: <date>
Current task: <task name>
Status: <what was done, what is not yet done>
Branch: <current branch name>
Next action: <exactly what to do when resuming>
```

---

4. ASK where to save the IMPLEMENTATION.md if not specified. Default: project root or alongside task docs if no project root is clear yet.

5. After saving, inform the user:
   - How many tasks were found and indexed
   - The safe serial execution order
   - Any tasks whose dependencies could not be resolved from the task docs (flag these)
