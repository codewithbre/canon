Verify that a task document is self-contained and executable by an
agent with no other context (no conversation history, no breakdown, no
other task docs - only the document itself and the codebase).

This skill has one responsibility: produce a verification report.
It does not modify anything.

Target: [path to a task document or directory of task documents]

1. RESOLVE the target:
   - Single file: verify that one document.
   - Directory: verify every task document in it. If it's unclear which
     files are task documents (vs. overview/README/index files), ask.
   - Empty: ask which document or directory to verify.

2. For each task document, spawn a task-doc-verifier agent with the
   document's absolute path. It has no access to this conversation - it
   reads the document cold, exactly as the eventual execution agent would.
   If an overview.md exists in the same directory, pass its path to the
   verifier as well. The verifier should use the overview's Codebase
   Analysis section as its starting context rather than re-exploring the
   codebase from scratch.

3. Present a summary:
   - One line per document: filename + confidence rating
   - Full gap list per document, grouped by recommended action
     (REWRITE / ADD CONTEXT / ASK AUTHOR)

4. After all documents are assessed, add a brief Process observations
   section if any of the following apply — omit it if none do:
   - Gaps required significant codebase re-exploration: the overview's
     Codebase Analysis was incomplete.
   - Multiple JUDGMENT-REQUIRED criteria across documents: the breakdown
     did not resolve enough detail before task docs were written.
   - Dependencies were not described concretely: task ordering in the
     breakdown was not sufficiently specific.
   These are not fixes for the current docs. They are signals for
   improving future breakdowns.

5. Do not edit any task document. If asked to act on a finding and the
   fix is ambiguous, surface the ambiguity and wait for direction.
