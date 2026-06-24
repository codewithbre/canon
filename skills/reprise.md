Generate a retrospective report for a completed canon workflow run and post it as a GitHub issue.

**If this project uses programmatic orchestration (Octave or similar):**
A `skills/reprise-api.md` variant is required alongside this skill.
The `-api` variant produces the retrospective body directly with no bash
commands, no narration, and no preamble. This skill's step 5 (`gh issue create`)
is incompatible with API use — the model outputs those commands as content
rather than executing them. Verify the `-api` variant exists before
beginning implementation.

This is the 7th and final step in the workflow:
/breakdown → /write-task → /verify-task → /create-tracker → /implement → /create-pr → /reprise

$ARGUMENTS should be the path to the project's IMPLEMENTATION.md, or the GitHub repo (owner/repo).
If neither is provided, look for IMPLEMENTATION.md in the current directory.

1. READ the IMPLEMENTATION.md to extract:
   - The original high-level goal (from the session notes or first task entry)
   - The full task list and which tasks are marked complete
   - The dependency graph and execution order

2. READ each completed task document to understand:
   - What was planned (Outcome, Scope)
   - What was delivered (Acceptance Criteria met)

3. IDENTIFY shortcomings by looking for evidence of:
   - Tasks that required re-scoping mid-implementation
   - Acceptance criteria that needed to be adjusted after the fact
   - Things that fell outside any task's defined scope (gap work done ad hoc)
   - Skills that produced incomplete or ambiguous output requiring human correction
   - Human interventions beyond the expected review gates
   - Tasks that were blocked and had to be reordered

4. PRODUCE a retrospective report with exactly these sections:

---

## Original Goal
[The high-level goal from the /breakdown invocation]

## What Was Delivered
[Bullet list of completed tasks with their branch/PR reference]

## What Worked Well
[Specific things the skill chain handled cleanly]

## Shortcomings
[Specific gaps, failures, or friction. Name the task doc, skill, or step. Vague feedback is not actionable.]

## Suggested Improvements
For each shortcoming, classify as one of:
- **New skill:** A skill that would have handled this
- **Skill update:** A specific change to an existing skill prompt
- **Workflow change:** A change to ordering or gating
- **Out of scope:** Belongs to a different agent

## Metrics
[Token usage, cost, most expensive step, wall-clock time if available from LangSmith. Otherwise note that observability data was not collected.]

---

5. POST as a GitHub issue:
   - Title: `Reprise: [original goal summary]`
   - Label: `reprise`
   - Body: the full report

   If the `reprise` label does not exist: `gh label create reprise --color "#8b5cf6" --description "Post-run retrospective"`
   Then: `gh issue create --title "Reprise: [goal]" --label reprise --body "..."`

6. OUTPUT the issue URL.

Do not modify any task documents, IMPLEMENTATION.md, or source files. This skill is read-only except for the GitHub issue it creates.
