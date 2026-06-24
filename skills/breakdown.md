Break down the work described below into executable tasks.
Do not implement anything. This is planning only.

For the work described:

1. ANALYZE the codebase to understand the current architecture,
   patterns, and relevant files.

2. IDENTIFY the functional changes required. Each task must be:
   - A single functional change (one intent, one output).
   - Scoped to a specific location in the codebase.
   - Small enough to complete and verify in a single pass.
   - Independent or with clearly stated dependencies on other tasks.

3. For each task, provide:
   - TASK: A clear one-line description of the functional change.
   - INTENT: What should be different when this task is done? What is the expectation of the outcome?
   - LOCATION: What files/components are likely in scope.
   - DEPENDS ON: Which other tasks must be completed first (if any).
   - ACCEPTANCE: How to verify this task is complete (observable output).
   - CONFIDENCE: HIGH / MEDIUM / LOW — how well scoped this task is.
     HIGH: intent, location, and acceptance criteria are all clear.
     MEDIUM: intent is clear but location or acceptance criteria need more definition.
     LOW: scope or acceptance criteria are unclear. A LOW rating surfaces a gap
     that needs human direction before a task document can be written. State
     what is unclear and what decision is needed. Wait for the human to resolve
     it before proceeding to the write-task skill for that task.

4. ORDER the tasks by dependency, then by logical sequence.

5. FLAG any areas where you lack enough information to define
   a task. State what decisions are needed before that task can be scoped.

Do not begin work on any task. Present the breakdown, then provide
a readiness summary:

READY TO PROCEED (HIGH / MEDIUM confidence):
  - Task N: <title>

NEEDS DIRECTION (LOW confidence):
  - Task N: <title> — <what needs to be resolved before this can proceed>

Tasks that are READY can proceed to the write-task skill immediately
after approval. Tasks that NEED DIRECTION must be resolved — clarified
to HIGH or MEDIUM, or removed — before write-task can be invoked for them.

Wait for review.

After the breakdown is approved:

1. DERIVE a directory name from the work description — short, kebab-case,
   and descriptive of the intent (e.g. tasks/projects-page/,
   tasks/auth-flow/). Confirm the location and name if unclear.

2. WRITE an overview document to that directory named overview.md:

   # <Feature / Intent Title>

   ## Intent
   What this body of work delivers and why it matters.

   ## Codebase Analysis
   Key files, patterns, and conventions discovered during breakdown.
   This section exists so the write-task and verify-task skills can
   reference it instead of re-exploring the codebase from scratch.

   ### Key files
   | File | Purpose |
   |------|---------|
   | `path/to/file` | What it does and why it matters for this work |

   ### Patterns
   Naming conventions, import patterns, component structure, data flow —
   anything an implementing agent needs to follow.

   ### Conventions
   Project-specific rules (typing, styling, accessibility, testing) that
   apply to this body of work.

   ## Tasks
   | # | Task | Confidence | Depends On |
   |---|------|------------|------------|
   | 1 | <task title> | HIGH | — |

   ## Execution order
   Safe serial order for implementation.

   ## Flags
   Any decisions or unknowns surfaced during breakdown.

3. For each approved task, use the write-task skill and save each document
   to the same directory as the overview. LOW confidence tasks that have
   not been resolved are captured as flags in the overview — do not invoke
   write-task for them until the human provides direction.

Work to break down: [describe the feature, epic, or task to plan]
