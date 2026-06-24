Implement the task document at the path provided. This skill has one
responsibility: produce working code that satisfies the stated outcome.

$ARGUMENTS should be the path to the task document.

This skill does not:
- Make decisions not already in the task document
- Modify files not listed in the Scope table
- Interpret, extend, or improve on the stated requirements
- Ask clarifying questions — if something is genuinely ambiguous, stop and
  state exactly what is blocking before writing any code

1. READ the task document in full before doing anything else:
   - Outcome: what will be true when this is done
   - Scope: the exact files to create or modify
   - Out of Scope: what must not change
   - Acceptance Criteria: the verifiable conditions to meet

2. RESTATE before writing any code:
   ```
   TASK: <title>
   WILL CHANGE:
     - <file> — <what changes>
   WILL NOT TOUCH:
     - <out of scope items>
   CRITERIA:
     1. <criterion>
     2. <criterion>
   ```
   Do not proceed past this point if anything in the task document is
   contradictory or unresolvable from the codebase alone.

3. READ only the files listed in the Scope table plus any direct imports
   needed to understand a type or interface those files depend on.
   Do not explore the broader codebase.

4. IMPLEMENT by creating or modifying only the files in Scope.
   Follow the patterns, naming conventions, and types already present
   in those files. Do not introduce new dependencies or abstractions
   not specified in the task document.

5. RUN the project's checks after writing code. Use commands from the
   task document or CLAUDE.md if present. If neither specifies them,
   use the most common for the project's stack:
   - TypeScript: `pnpm typecheck` or `tsc --noEmit`
   - Formatter: `pnpm format` or `prettier --write`
   If checks fail, fix the errors before reporting complete.

6. VERIFY each acceptance criterion against the actual output:
   - PASS: the criterion is demonstrably met
   - FAIL: state what is missing and fix it
   Do not report complete until every criterion is PASS.

7. REPORT on completion:
   ```
   FILES CHANGED:
     - <file> — <created / modified>
   CRITERIA:
     1. <criterion> — PASS
     2. <criterion> — PASS
   CHECKS: <commands run and result>
   ```
