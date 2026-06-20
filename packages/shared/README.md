# Shared

Common code used by both Fugue and Coda: TypeScript types, LangChain + LangSmith LLM client, GitHub API client, skill loader.

Part of the [Orchestra](../../README.md) monorepo.

## Contents (planned)

- `src/types.ts` — shared types: `Task`, `TaskDoc`, `VerifyResult`
- `src/llm.ts` — configured `ChatAnthropic` instance with LangSmith tracing
- `src/github.ts` — `@octokit/rest` client: branch, commit, PR operations
- `src/skills.ts` — loads `skills/*.md` files as system prompt strings

## Status

Planning. Will be extracted from `packages/fugue/runtime/` once the runtime is built.
