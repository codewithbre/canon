# Task 04 — Breakdown Tool

## Outcome

`packages/fugue/runtime/src/tools/breakdown.ts` exports `runBreakdown(taskDescription: string, codebaseSummary: string): Promise<Task[]>`. It calls Claude using the breakdown skill as system prompt and returns a typed array of tasks parsed via Zod structured output. Each `Task` has `title`, `intent`, `location`, `dependsOn`, and `acceptance` fields.

## Why

The breakdown tool is the entry point of the entire canon workflow. It is the first call the orchestrator makes, and the quality of its output determines the quality of everything that follows. Wrapping it as a typed async function with Zod validation means the orchestrator receives a structured task list, not raw text to parse.

## Context

### Repository state

Tasks 01, 02, and 03 are complete:
- `packages/fugue/runtime/src/skills.ts` exports `loadSkill(name: SkillName): string`
- `packages/fugue/runtime/src/llm.ts` exports `createLLM(): ChatAnthropic`
- `packages/fugue/runtime/package.json` has `zod` installed

### Task type

Define in `packages/fugue/runtime/src/types.ts` (create if it doesn't exist):

```typescript
import { z } from 'zod';

export const TaskSchema = z.object({
  title: z.string(),
  intent: z.string(),
  location: z.string(),
  dependsOn: z.array(z.string()),
  acceptance: z.string(),
});

export type Task = z.infer<typeof TaskSchema>;

export const TaskListSchema = z.object({
  tasks: z.array(TaskSchema),
});
```

### Implementation pattern

```typescript
import { loadSkill } from '../skills.js';
import { createLLM } from '../llm.js';
import { TaskListSchema, type Task } from '../types.js';
import { SystemMessage, HumanMessage } from '@langchain/core/messages';

export async function runBreakdown(
  taskDescription: string,
  codebaseSummary: string
): Promise<Task[]> {
  const llm = createLLM().withStructuredOutput(TaskListSchema);
  const systemPrompt = loadSkill('breakdown');

  const response = await llm.invoke([
    new SystemMessage(systemPrompt),
    new HumanMessage(`Task: ${taskDescription}\n\nCodebase context:\n${codebaseSummary}`),
  ]);

  return response.tasks;
}
```

## Scope

| File | Action |
|---|---|
| `packages/fugue/runtime/src/types.ts` | Create — `Task`, `TaskSchema`, `TaskListSchema` |
| `packages/fugue/runtime/src/tools/breakdown.ts` | Create — `runBreakdown()` |

## Out of Scope

- Do not create tools for other skills — that is Tasks 05, 06.
- Do not create the orchestrator — that is Task 08.
- Do not add retry logic for failed API calls.

**Dependency:** Tasks 01, 02, and 03 must be complete.

## Acceptance Criteria

1. `pnpm typecheck --filter fugue-runtime` passes.
2. `runBreakdown('Add a dark mode toggle', 'React app with Tailwind CSS')` with valid API keys returns an array of `Task` objects, each with all five required fields populated.
3. The returned array has at least one task.
4. `Task.dependsOn` is an array of strings (empty array if no dependencies).
5. The LangSmith trace for the call shows the breakdown skill as the system prompt content.
