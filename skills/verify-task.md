Verify that a task document is self-contained and executable by an
agent with no other context (no conversation history, no breakdown, no
other task docs - only the document itself and the codebase).

This command does not modify anything. It produces a report only.

Target: $ARGUMENTS

1. RESOLVE the target:
   - Single file: verify that one document.
   - Directory: verify every task document in it. If it's unclear which
     files are task documents (vs. README/index files), ask.
   - Empty: ask which document or directory to verify.

2. For each task document, spawn a `task-doc-verifier` agent with the
   document's absolute path. It has no access to this conversation - it
   reads the document cold, exactly as the eventual execution agent would.

3. Present a summary:
   - One line per document: filename + confidence rating
   - Full gap list per document, grouped by recommended action
     (REWRITE / ADD CONTEXT / ASK AUTHOR)

4. Do not edit any task document yourself. If the user asks you to act
   on a finding and the fix itself is ambiguous, apply the Ambiguity
   Resolution protocol rather than guessing.
