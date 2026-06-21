# Task 02 — Skill Loader

## Outcome

`packages/fugue/runtime/src/skills.ts` exports `loadSkill(name: SkillName): string` that reads the corresponding `skills/*.md` file from the orchestra repo root and returns its full content as a string. A `SkillName` type union covers all seven valid skills. This is the function all tool wrappers call to get their system prompt.

## Why

Every tool in the runtime wraps a skill definition from `skills/*.md`. Without a loader, each tool would hardcode its own path resolution or duplicate the file-read logic. Centralizing it here means skill file paths are defined once, and adding a new skill only requires updating `SkillName`.

## Context

### Repository state

Task 01 is complete. `packages/fugue/runtime/` exists with `package.json`, `tsconfig.json`, and an empty `src/index.ts`.

The seven skill files live at the monorepo root:
```
skills/
├── breakdown.md
├── write-task.md
├── verify-task.md
├── create-tracker.md
├── implement.md
├── create-pr.md
└── reprise.md
```

The runtime is at `packages/fugue/runtime/`. To reach the root `skills/` directory from there, the path is `../../../skills/`.

### Node.js path resolution

Use `path.join` and `fileURLToPath` for ESM-compatible path resolution:

```typescript
import { readFileSync } from 'node:fs';
import { fileURLToPath } from 'node:url';
import path from 'node:path';

const __dirname = path.dirname(fileURLToPath(import.meta.url));
const SKILLS_DIR = path.join(__dirname, '..', '..', '..', '..', 'skills');
```

From `src/skills.ts` (inside `packages/fugue/runtime/src/`), four `..` levels reach the monorepo root.

## Scope

| File | Action |
|---|---|
| `packages/fugue/runtime/src/skills.ts` | Create |

## Out of Scope

- Do not copy skill file content into the source — always read from disk at call time.
- Do not modify the `skills/*.md` files.
- Do not create any LLM client or tool wrapper — that is Tasks 03-06.

**Dependency:** Task 01 must be complete (`runtime/package.json` and `tsconfig.json` must exist).

## Acceptance Criteria

1. `pnpm typecheck --filter fugue-runtime` passes with no errors.
2. `loadSkill('breakdown')` returns the full text content of `skills/breakdown.md` as a string.
3. `loadSkill('write-task')` returns the full text of `skills/write-task.md`.
4. `loadSkill('reprise')` returns the full text of `skills/reprise.md`.
5. TypeScript type `SkillName` is a string union of all seven skill names: `'breakdown' | 'write-task' | 'verify-task' | 'create-tracker' | 'implement' | 'create-pr' | 'reprise'`.
6. Passing an invalid skill name to `loadSkill` is a TypeScript compile error (not a runtime error).
