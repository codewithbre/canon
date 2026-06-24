Create a pull request summary for the current changes.
This skill has one responsibility: produce a summary that proves
an intention was met. Do not push, create, or modify the PR unless
explicitly instructed. This is documentation only.

1. ANALYZE the current branch changes against the base branch.
   If the changes do not represent a single coherent functional
   change, flag this before producing a summary.

2. PRODUCE the following:

   ## Title
   A concise statement of what is now true that wasn't before.
   Describe the outcome. Not the mechanism.

   ## Intention
   What was the goal? What problem was being solved or what
   capability was being added?

   ## Outcome
   What is now different? Describe the end state as an observable
   result.

   ## Value
   What does this change enable, unblock, or improve?

   ## Changes
   A high-level summary of what changed. Only call out specific
   files if they were the primary source of change or are
   significant to understanding the PR.

   ## Verification
   How to confirm the outcome was achieved. Map to the task's
   acceptance criteria where they exist.

   ## Scope Confirmation
   - [ ] All changes are directly related to the stated intention.
   - [ ] No unrelated refactoring is included.
   - [ ] No new dependencies were added (or additions were approved).
   - [ ] All code is strongly typed with no `any` or lint suppressions.
   - [ ] Each commit represents a single functional change.

3. After completing step 2, verify: does the Intention align with
   the Outcome? Does the Outcome justify the Value claim? Update
   if not.

Target branch: [branch to compare against the base branch]
