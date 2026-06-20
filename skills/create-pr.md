Create a pull request summary for the current changes. Do not
push or create the PR unless instructed to — produce the summary 
with github cli commands.

1. ANALYZE the current branch changes against the base branch.

2. PRODUCE the following:

   ## Title
   A concise title following conventional commit format:
   type(scope): description
   Types: feat, fix, refactor, test, docs, chore

   ## Summary
   2-3 sentences describing what this PR does and why.

   ## Changes
   A list of every file modified, created, or deleted, grouped
   by category:
   - State the intention of the code change and the output/result of the change
   - semantically organize by related code changes vs file by file list

   ## How to Verify
   Step-by-step instructions for reviewing and testing this PR.

   ## Scope Confirmation
   - [ ] All changes are directly related to the stated task.
   - [ ] No unrelated refactoring is included.
   - [ ] No new dependencies were added (or additions were approved).
   - [ ] All code is strongly typed with no `any` or lint suppressions.
   - [ ] Each commit represents a single functional change.

   ## Known Limitations / Follow-up
   Any items that are out of scope but worth noting for future work.

3. If the changes do not look like a coherent single functional
   change, flag this. State what appears to be bundled together
   and suggest how to split it.

4. Ensure that properly completed step 2, re-review and update if needed

Target branch: $ARGUMENTS