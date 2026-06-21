# Task 07 — GitHub Client

## Outcome

`packages/fugue/runtime/src/github.ts` exports four functions — `getDefaultBranch`, `createBranch`, `commitFile`, and `openPR` — that wrap `@octokit/rest` to perform the GitHub operations the orchestrator needs. All functions are authenticated via `GITHUB_TOKEN` from the environment.

## Why

The orchestrator needs to create branches for task docs, commit the doc files, open PRs for implementations, and post issue comments for human-in-the-loop gates. Centralizing all GitHub API calls here means authentication, error handling, and the `owner/repo` parsing happen once.

## Context

### Repository state

Task 01 is complete. `@octokit/rest` is declared as a dependency in `packages/fugue/runtime/package.json` and installed.

### Function signatures

```typescript
// Parse "owner/repo" string into { owner, repo }
export function parseRepo(repo: string): { owner: string; repo: string }

// Get the SHA of the default branch HEAD
export async function getDefaultBranch(repo: string): Promise<{ branch: string; sha: string }>

// Create a new branch from a base SHA
export async function createBranch(repo: string, branchName: string, baseSha: string): Promise<void>

// Commit a single file to a branch
export async function commitFile(
  repo: string,
  branch: string,
  filePath: string,    // e.g. ".claude/tasks/task-01-add-feature.md"
  content: string,     // file content (not base64 — function handles encoding)
  message: string,
): Promise<void>

// Open a PR from a branch to the default branch
export async function openPR(
  repo: string,
  branch: string,
  title: string,
  body: string,
  draft?: boolean,     // default: true
): Promise<{ url: string; number: number }>

// Post a comment on an issue or PR
export async function postComment(
  repo: string,
  issueNumber: number,
  body: string,
): Promise<void>
```

### Octokit setup

```typescript
import { Octokit } from '@octokit/rest';

function getOctokit(): Octokit {
  const token = process.env.GITHUB_TOKEN;
  if (!token) throw new Error('GITHUB_TOKEN is required');
  return new Octokit({ auth: token });
}
```

Content passed to `commitFile` must be base64-encoded before the API call:
```typescript
const encodedContent = Buffer.from(content, 'utf-8').toString('base64');
```

## Scope

| File | Action |
|---|---|
| `packages/fugue/runtime/src/github.ts` | Create — all six functions |

## Out of Scope

- Do not implement any logic that reads from GitHub (fetching PR status, polling comments) — that is part of the orchestrator's human-in-the-loop gate.
- Do not create GitHub Actions workflow files here — that is Task 10.

**Dependency:** Task 01 must be complete.

## Acceptance Criteria

1. `pnpm typecheck --filter fugue-runtime` passes.
2. `getDefaultBranch('codewithbre/orchestra')` returns an object with `branch: 'main'` and a non-empty `sha` string.
3. `createBranch('codewithbre/orchestra', 'test-branch-delete-me', sha)` creates a branch visible on GitHub. (Clean up after testing.)
4. `commitFile('codewithbre/orchestra', 'test-branch-delete-me', 'test.txt', 'hello', 'test commit')` commits the file to the branch.
5. `openPR(...)` opens a draft PR and returns `{ url, number }`.
6. All functions throw a clear error when `GITHUB_TOKEN` is not set.
