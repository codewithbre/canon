Create an IMPLEMENTATION.md tracker document from a set of completed task documents.
This is the 4th step in the planning workflow:
/breakdown -> /write-task -> /verify-task -> /create-tracker

This command has one responsibility: produce a lean status tracker that lets
any agent or developer orient and resume work cold. Do not implement anything.

$ARGUMENTS should be the path to the directory containing task documents (e.g. `.claude/tasks/feature-name/`).
If no path is provided, look for task documents in the current project's `.claude/tasks/` directory.
If still unclear, ask.

1. SCAN the target directory and read every task document found.
   Skip overview.md, README files, index files, and non-task files.
   Task documents are identified by having the standard sections: Outcome, Why, Context, Scope, Out of Scope, Acceptance Criteria.

2. EXTRACT from each task document:
   - Task title (from the # heading)
   - One-line summary of what the task does (from Outcome, condensed)
   - Branch name (from the task doc if stated, otherwise derive from the filename: task-01-name.md -> task-01-name)
   - Dependencies (from "Depends on" or "Dependency" sections)

3. PRODUCE an IMPLEMENTATION.md with exactly these sections:

---

# <Project Name> - Implementation Tracker

<One sentence describing what this body of work delivers.>

**Overview:** `.claude/tasks/<dir>/overview.md`

---

## Current state

```
Status: not started
Active task: none
Branch: none
Next action: start Task 1 - <task title>
Last updated: <date>
```

Update this block at the start and end of every session.
A cold agent reading this file should be able to orient from this block alone.

---

## Task checklist

Check a task off only when its PR is merged into main.
Read the linked task document before starting each task.

[For each task, produce a block like:]

### <Task title>
**Task doc:** `.claude/tasks/<dir>/<filename>`
**Branch:** `<branch-name>`
**Summary:** <one-line summary>
**Depends on:** <task title, or "Nothing">

- [ ] Complete (PR merged)

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

## File locations

| Artifact | Path |
|----------|------|
| Overview | `.claude/tasks/<dir>/overview.md` |
| Task docs | `.claude/tasks/<dir>/` |

---

4. ASK where to save the IMPLEMENTATION.md if not specified.
   Default: alongside the task docs in the same directory.

5. After saving, inform the user:
   - How many tasks were found and indexed
   - The safe serial execution order
   - Any tasks whose dependencies could not be resolved from the task docs (flag these)
