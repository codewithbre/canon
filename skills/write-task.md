Write a standalone task document for the functional change described
below. This document is the implementation guide. It is the only
instruction the implementing agent will receive beyond the codebase.
It must be self-contained: another agent, given only this document
and the codebase, must be able to complete the task without any other
context (this conversation, a breakdown list, or other task documents).

Do not implement anything. This is documentation only.

0. CHECK for an overview.md in the task directory before exploring the codebase.
   If one exists, read its Codebase Analysis section first. Use the key files,
   patterns, and conventions it documents as the starting context for this task.
   Only explore the codebase further for details specific to this task that are
   not already covered. This avoids redundant codebase re-exploration across tasks.

1. ANALYZE the codebase to confirm the task is valid per Task Integrity
   (single functional change, clear intent, clear location, clear
   output, verifiable in one pass). If it is not, stop and explain why
   instead of writing a document.

2. PRODUCE a markdown document with exactly these sections:

   # <Task Title>

   ## Outcome
   What will be different when this task is done. Describe the end
   state, not the steps to get there.

   ## Why
   The value or reason this change matters - what problem it solves
   or what it enables. This is the justification, not a restatement
   of the outcome.

   ## Context
   Everything an agent needs to know about the current codebase to
   start this task cold: relevant files, current behavior, existing
   patterns to follow, related types/constants/conventions. Write as
   if the reader has never seen this conversation.

   ## Scope
   The specific files/areas that will change.

   ## Out of Scope
   Explicit negative framing: what this task does NOT include, what
   must NOT change, and adjacent work that belongs to a different
   task. If this task depends on another task's output, state that
   dependency explicitly here - otherwise assume no dependency.

   ## Acceptance Criteria
   Observable, verifiable conditions that confirm the outcome has
   been achieved.

3. Tasks must be independent of each other. Do not write "see Task 3"
   or rely on shared context from a breakdown - if information from
   another task is needed, restate it here.
   Tasks are executed by isolated agents with no shared context beyond
   the codebase and this document. A different agent may implement each
   task with no awareness of the others.
   If this task depends on a prior change, restate what that change
   produces and where to find it in the codebase.

4. Each invocation of this skill produces exactly one document,
   corresponding to one task. By convention, one task corresponds to
   one PR's worth of work - name and scope the document accordingly
   (e.g. tasks/feature-name/task-01-name.md).

5. After producing the document, provide a confidence rating before saving:
   - CONFIDENCE: HIGH / MEDIUM / LOW
     HIGH: the document is complete and an agent can execute it without ambiguity.
     MEDIUM: one or more sections may require inference from the codebase to fill gaps.
     LOW: significant gaps exist that should be resolved before implementation begins.
   State the rating and the specific reason. A low rating is not a failure —
   it surfaces where human judgment is needed before the task proceeds.

6. ASK where to save the document if not already established by the breakdown.
   Do not write the file until a location is confirmed.

After the document is saved, use the verify-task skill on the saved file
path to confirm it is self-contained and executable by a cold agent.

Task to document: [the approved task from the breakdown]
