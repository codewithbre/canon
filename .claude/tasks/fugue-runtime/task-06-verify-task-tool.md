# Task 06 — Verify-Task Tool

## Outcome

`packages/fugue/runtime/src/tools/verify-task.ts` exports `runVerifyTask(doc: TaskDoc): Promise<VerifyResult>` where `VerifyResult` is `{ confidence: 'HIGH' | 'MEDIUM' | 'LOW'; gaps: string[] }`. It calls Claude with the verify-task skill as system prompt, passing the task doc content, and returns a structured assessment of whether the doc is self-contained and executable by an isolated agent.

## Why

The verify-task tool runs in parallel after all write-task calls complete. A `LOW` confidence result or a non-empty `gaps` array signals that the orchestrator should surface the issue to the human before proceeding to implementation. This gate prevents bad task docs from reaching the implement step.

## Context

### Repository state

Tasks 01-05 are complete:
- `src/skills.ts` exports `loadSkill()`
- `src/llm.ts` exports `createLLM()`
- `src/types.ts` exports `Task`, `TaskDoc`

### VerifyResult type

Add to `packages/fugue/runtime/src/types.ts`:

```typescript
export const VerifyResultSchema = z.object({
  confidence: z.enum(['HIGH', 'MEDIUM', 'LOW']),
  gaps: z.array(z.string()),
});

export type VerifyResult = z.infer<typeof VerifyResultSchema>;
```

### Implementation pattern

```typescript
export async function runVerifyTask(doc: TaskDoc): Promise<VerifyResult> {
  const llm = createLLM().withStructuredOutput(VerifyResultSchema);
  const systemPrompt = loadSkill('verify-task');

  return llm.invoke([
    new SystemMessage(systemPrompt),
    new HumanMessage(
      `Verify the following task document:\n\nFilename: ${doc.filename}\n\n${doc.content}`,
    ),
  ]);
}
```

## Scope

| File | Action |
|---|---|
| `packages/fugue/runtime/src/types.ts` | Update — add `VerifyResult`, `VerifyResultSchema` |
| `packages/fugue/runtime/src/tools/verify-task.ts` | Create — `runVerifyTask()` |

## Out of Scope

- Do not modify the task doc based on verification results — the orchestrator decides what to do with gaps.
- Do not block or throw on LOW confidence — return the result and let the orchestrator handle it.

**Dependency:** Tasks 01, 02, 03, and 05 must be complete (`TaskDoc` type must exist).

## Acceptance Criteria

1. `pnpm typecheck --filter fugue-runtime` passes.
2. `runVerifyTask(wellFormedDoc)` returns `{ confidence: 'HIGH', gaps: [] }`.
3. `runVerifyTask(incompleteDoc)` — where `incompleteDoc` is missing Acceptance Criteria — returns `{ confidence: 'LOW', gaps: ['Missing acceptance criteria'] }` or similar.
4. The function can be called in parallel for multiple docs without error.
5. The LangSmith trace shows the verify-task skill as the system prompt.
