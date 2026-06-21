# Task 03 — LangChain + LangSmith LLM Client

## Outcome

`packages/fugue/runtime/src/llm.ts` exports `createLLM()` returning a configured `ChatAnthropic` instance, and `invokeLLM()` — a wrapper that calls `llm.invoke()` and throws `TruncatedOutputError` if `stop_reason` is `max_tokens`. Every tool in the runtime calls `invokeLLM()` instead of `llm.invoke()` directly, so truncated outputs never silently pass as successes. When `LANGCHAIN_TRACING_V2=true` and `LANGCHAIN_API_KEY` are set, all calls are automatically traced in LangSmith under the `fugue` project.

## Why

Every tool in the runtime makes LLM calls. A single `createLLM()` function means all tools share the same model configuration and tracing setup. Changing the model or tracing config in one place affects the entire runtime.

## Context

### Repository state

Task 01 is complete. `@langchain/anthropic` and `langsmith` are declared as dependencies in `packages/fugue/runtime/package.json` and installed.

### LangSmith tracing

LangSmith tracing is automatic when these env vars are set:
```
LANGCHAIN_TRACING_V2=true
LANGCHAIN_API_KEY=<your key>
LANGCHAIN_PROJECT=fugue
```

No code changes needed for tracing — `@langchain/anthropic` detects these env vars at import time and instruments all `.invoke()` calls automatically.

### Prompt caching support

The `ChatAnthropic` instance returned by `createLLM()` supports Anthropic prompt caching without additional client configuration. Tool wrappers and the orchestrator activate caching by placing stable content first in the message array and tagging it with `cache_control: { type: 'ephemeral' }`:

```typescript
import { HumanMessage } from '@langchain/core/messages';

const response = await llm.invoke([
  new HumanMessage({
    content: [
      {
        type: 'text',
        text: skillPrompt,           // stable — cached after first call
        cache_control: { type: 'ephemeral' },
      },
      {
        type: 'text',
        text: taskSpecificContext,   // variable — not cached
      },
    ],
  }),
]);
```

Cache TTL is 5 minutes. Skill prompts loaded from `skills/*.md` are the primary candidates — they are identical across all parallel write-task and verify-task calls in a single run. At 10 parallel write-task calls, caching the skill prompt collapses the overhead from ~5,000 tokens to near-zero on calls 2–N.

`createLLM()` itself does not set cache headers — that is the caller's responsibility. The function only needs to return a client that passes `cache_control` blocks through to the API, which `@langchain/anthropic` does by default.

### Output truncation check

The Anthropic API returns HTTP 200 with `stop_reason: "max_tokens"` when the model hits its output limit. This looks like success. A truncated task doc that silently passes into verify-task is a worse outcome than a loud failure.

Every tool in the runtime must call `invokeLLM()` — not `llm.invoke()` — so truncation is caught centrally:

```typescript
import { ChatAnthropic } from '@langchain/anthropic';
import type { BaseMessage } from '@langchain/core/messages';
import type { AIMessage } from '@langchain/core/messages';

export class TruncatedOutputError extends Error {
  constructor(step: string) {
    super(`LLM output truncated at step "${step}" (stop_reason: max_tokens)`);
    this.name = 'TruncatedOutputError';
  }
}

export async function invokeLLM(
  llm: ChatAnthropic,
  messages: BaseMessage[],
  step: string,
): Promise<AIMessage> {
  const response = await llm.invoke(messages);
  if (response.response_metadata?.['stop_reason'] === 'max_tokens') {
    throw new TruncatedOutputError(step);
  }
  return response;
}
```

### Implementation

```typescript
import { ChatAnthropic } from '@langchain/anthropic';

export function createLLM(): ChatAnthropic {
  const apiKey = process.env.ANTHROPIC_API_KEY;
  if (!apiKey) throw new Error('ANTHROPIC_API_KEY is required');

  return new ChatAnthropic({
    model: process.env.ANTHROPIC_MODEL ?? 'claude-opus-4-8',
    temperature: 0,
    anthropicApiKey: apiKey,
  });
}
```

Temperature 0 for all calls — the runtime needs deterministic, structured outputs, not creative variation.

## Scope

| File | Action |
|---|---|
| `packages/fugue/runtime/src/llm.ts` | Create — exports `createLLM`, `invokeLLM`, `TruncatedOutputError` |

## Out of Scope

- Do not create any tool or orchestrator logic — that is Tasks 04-08.
- Do not hardcode a specific model version as a constant — read from `ANTHROPIC_MODEL` env var with a sensible default.
- Do not add retry logic here — retry and rate-limit handling lives in the orchestrator (Task 08).

**Dependency:** Task 01 must be complete.

## Acceptance Criteria

1. `pnpm typecheck --filter fugue-runtime` passes.
2. `createLLM()` throws a clear error when `ANTHROPIC_API_KEY` is not set.
3. `createLLM()` returns a `ChatAnthropic` instance when the key is present.
4. With `LANGCHAIN_TRACING_V2=true` and a valid `LANGCHAIN_API_KEY`, calling `invokeLLM(llm, [...], 'test')` produces a trace in the LangSmith `fugue` project dashboard showing input tokens, output tokens, and cost.
5. Without `LANGCHAIN_TRACING_V2`, the same call completes without error and produces no trace.
6. A `HumanMessage` with a `content` array containing a block with `cache_control: { type: 'ephemeral' }` can be passed to `invokeLLM()` without a type error.
7. `invokeLLM()` throws `TruncatedOutputError` when the response has `stop_reason === 'max_tokens'`. It does not throw for any other stop reason.
