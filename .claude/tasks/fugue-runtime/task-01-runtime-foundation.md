# Task 01 — Fugue Runtime Project Foundation

## Outcome

`packages/fugue/runtime/` exists as a valid TypeScript ESM project. `pnpm install` from the monorepo root installs its dependencies. `pnpm typecheck --filter fugue-runtime` passes with no source errors. The folder structure matches the design in `packages/fugue/docs/runtime-design.md`.

## Why

Every subsequent runtime task depends on this scaffold existing. Without a valid `package.json` and `tsconfig.json`, no dependencies can be installed and no TypeScript can resolve. This is the prerequisite for all other runtime tasks.

## Context

### Repository state

Monorepo at `C:\code\personal\canon` (remote: `github.com/codewithbre/orchestra`). pnpm workspace covers `packages/*` via `pnpm-workspace.yaml`.

```
packages/fugue/
├── README.md
├── docs/
│   ├── runtime-design.md    ← defines the target structure
│   └── token-efficiency.md
└── runtime/                 ← CREATE THIS (currently empty directory)
```

### Target structure

```
packages/fugue/runtime/
├── package.json
├── tsconfig.json
├── .env.example
└── src/
    ├── index.ts             ← empty barrel
    └── tools/               ← empty directory
```

### package.json

```json
{
  "name": "fugue-runtime",
  "version": "0.1.0",
  "private": true,
  "type": "module",
  "scripts": {
    "typecheck": "tsc --noEmit",
    "start": "node --env-file=.env src/index.js"
  },
  "dependencies": {
    "@langchain/anthropic": "latest",
    "@langchain/core": "latest",
    "langsmith": "latest",
    "@octokit/rest": "latest",
    "zod": "latest"
  },
  "devDependencies": {
    "typescript": "^5.7.3",
    "@types/node": "^22.0.0"
  }
}
```

### tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["ES2022"],
    "module": "ESNext",
    "moduleResolution": "Bundler",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "skipLibCheck": true,
    "noEmit": true,
    "esModuleInterop": true,
    "isolatedModules": true
  },
  "include": ["src/**/*.ts"]
}
```

### .env.example

```
ANTHROPIC_API_KEY=
LANGCHAIN_API_KEY=
LANGCHAIN_TRACING_V2=true
LANGCHAIN_PROJECT=fugue
GITHUB_TOKEN=
```

## Scope

| File | Action |
|---|---|
| `packages/fugue/runtime/package.json` | Create |
| `packages/fugue/runtime/tsconfig.json` | Create |
| `packages/fugue/runtime/.env.example` | Create |
| `packages/fugue/runtime/src/index.ts` | Create (empty: `export {};`) |
| `packages/fugue/runtime/src/tools/.gitkeep` | Create |

## Out of Scope

- Do not implement any logic — empty files and config only.
- Do not create `packages/shared/` — shared code is extracted later.
- Do not modify anything outside `packages/fugue/runtime/`.

## Acceptance Criteria

1. `pnpm install` from the monorepo root completes without error.
2. `pnpm typecheck --filter fugue-runtime` exits 0.
3. `packages/fugue/runtime/package.json` declares all five dependencies listed above.
4. `packages/fugue/runtime/src/index.ts` exists and exports an empty module.
5. `packages/fugue/runtime/.env.example` exists with all five keys.
