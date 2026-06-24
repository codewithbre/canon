Orchestrate the full plan-to-pr development cycle for the work described.
This skill has one responsibility: sequence the planning workflow and
enforce the human judgment gate at the right moment. Do not implement
anything.

Human judgment is front-loaded at Step 1. The breakdown is where scope,
intent, and task boundaries are decided. Every step after formalizes and
verifies that decision. Do not re-litigate the breakdown in later steps.

Work to plan: [describe the feature, epic, or task to plan]

## Step 1: Breakdown
Use the breakdown skill on the described work.
Its sole responsibility is to decompose the work into a reviewable
task list. It does not write documents or make decisions.

Present the breakdown and STOP. This is the only gate where the plan
can change. Wait for explicit approval before proceeding.

## Step 2: Write task documents
Use the write-task skill for each approved task.
Its sole responsibility is to produce a self-contained document that
any agent can execute with no context beyond the codebase.

Confirm the save location once before writing the first document.
Write all documents without stopping. Only pause if a task cannot
be scoped from the approved breakdown, or if a task has LOW confidence
and needs human direction first. Surface it and wait before continuing.

## Step 3: Verify task documents
Use the verify-task skill on the task document directory.
Its sole responsibility is to confirm each document is executable
by an agent with no context beyond the document and the codebase.

If all documents pass, proceed without stopping.
If gaps are found, surface them and wait for direction.
Do not edit task documents.

## Step 4: Create implementation tracker
Use the create-tracker skill on the task document directory.
Its sole responsibility is to produce the sequenced implementation
tracker from the verified task documents.

Confirm save location and write. Planning is complete.

## Execution and feedback

Each task in the tracker is its own execution session.
The implementing agent's sole responsibility is to adhere to the
task document. The plan is already decided. There is nothing to
interpret or extend. Only to deliver.

After implementation, human UAT determines the next step:
- Outcome matches intention: use the create-pr skill to close the loop.
- Outcome falls short: use the feedback skill to capture the gap.
  The feedback skill appends a correction to the task document without
  modifying existing content. The correction is delivered before
  the create-pr skill is invoked.
