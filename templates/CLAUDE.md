# CLAUDE.md - [PROJECT NAME]

## What this project is

[One sentence describing what this project does and who it's for.]

---

## Primary Principle: Task Integrity

Every change to this codebase must be a single functional change with:
- A clear intent (what will be different when this is done)
- A clear location (exactly which files change)
- A clear output (what the deliverable looks like)
- A scope that can be verified in one pass

Do not accept tasks that require architectural decisions not already made, bundle multiple unrelated changes, or cannot be verified in a single pass.

---

## Workflow router

Use this table before beginning any work:

| Situation | Skill |
|---|---|
| Need to scope new work | `/breakdown` |
| Writing a task document | `/write-task` |
| Verifying task docs | `/verify-task` |
| Generating IMPLEMENTATION.md | `/create-tracker` |
| Implementing a scoped task | `/implement` |
| Writing a PR description | `/create-pr` |

Never begin implementation without a task document.

---

## Stack

[PROJECT-SPECIFIC: List language, framework, key libraries, package manager, deployment target.]

---

## Directory structure

[PROJECT-SPECIFIC: Brief map of where things live.]

```
src/
├── [describe key directories]
```

---

## Commands

[PROJECT-SPECIFIC: List the commands an agent needs to know.]

```bash
# Install
[install command]

# Type check
[typecheck command]

# Format
[format command]

# Test
[test command]

# Dev server / run
[dev command]
```

---

## Conventions

[PROJECT-SPECIFIC: Naming patterns, file organization, typing conventions, import style.]

---

## Do not

[PROJECT-SPECIFIC: Project-specific constraints. The following apply to all canon projects:]

- Do not begin implementation without a task document
- Do not make changes outside the scope defined in the task document
- Do not guess when requirements are ambiguous — surface the ambiguity and stop
- Do not commit or push without explicit instruction
- Do not add dependencies without approval
- Do not use `any`, `@ts-ignore`, or lint suppressions

---

## PR conventions

Every PR corresponds to one task document.

**Branch naming:** match the task doc filename without `.md` (e.g. `task-01-add-function`)

**Commit format:** `feat(task-01): short description of what this commit adds`

**PR body template:**
```
## What this does
[Restate the Outcome from the task doc in 1-2 sentences.]

## Why
[Restate the Why from the task doc in 1 sentence.]

## Changes
| File | Action |
|---|---|
| `path/to/file` | Created / Modified |

## Verification
- [ ] [Acceptance criterion 1]
- [ ] [Acceptance criterion 2]
- [ ] No files outside Scope were modified
```
