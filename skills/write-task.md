Write a standalone task document for the functional change described
below. This document must be self-contained: another agent, given only
the codebase and this document, must be able to complete the task
without any other context (this conversation, a breakdown list, or
other task documents).

**If this project uses programmatic orchestration (Octave or similar):**
A `skills/write-task-api.md` variant is required alongside this skill.
The `-api` variant produces the document directly with no questions,
no tool invocations, and no narration. This skill's "ASK where to save"
instruction (step 5) is incompatible with API use — the `-api` variant
removes it. Verify the `-api` variant exists before beginning implementation.

Do not implement anything. This is documentation only.

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
   Tasks are executed by isolated agents with no shared context beyond the codebase and this document. 
   Understand that a different agent may implement tasks and these agents will be agnostic of each
   other. Agents only have the context of the codebase and the task document created.
   If this task depends on a prior change, restate what that change produces and where to find it in the codebase.

4. Each invocation of this command produces exactly one document,
   corresponding to one task. By convention, one task corresponds to
   one PR's worth of work - name and scope the document accordingly
   (e.g. tasks/<task-name>.md).

   If the task creates an LLM client or tool wrapper, include a model
   selection table in the Context section:

   | Call | Model | Reason |
   |---|---|---|
   | planning/generation calls | `claude-sonnet-4-6` | Reasoning quality required |
   | evaluation/formatting calls | `claude-haiku-4-5-20251001` | Structural check only |

   If the task calls `llm.invoke()` directly, add this note to Context:
   "`response.content` may be a string or a `MessageContent` array.
   Use an `extractText(content)` helper that handles both forms."

5. ASK where to save the document (e.g. a tasks/ directory) if not
   specified. Do not write the file until a location is confirmed.

After the document is saved, if I want a fresh agent with no access to
this conversation to confirm the document is self-contained and
executable, use /verify-task on the saved file path.

Task to document: $ARGUMENTS
