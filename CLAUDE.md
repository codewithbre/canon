# CLAUDE.md - Canon

## What this repo is

Canon is a multi-agent coding workflow system. This repo documents the philosophy, defines the skill chain, and provides templates for adopting the workflow on any project.

This CLAUDE.md is for agents contributing to canon itself. If you are working on a project that has adopted canon, use that project's CLAUDE.md instead.

---

## Primary Principle: Task Integrity

Every change to this repo must be a single functional change with:
- A clear intent (what will be different when this is done)
- A clear location (exactly which files change)
- A clear output (what the deliverable looks like)
- A scope that can be verified in one pass

**Do not accept vague tasks.** "Improve the docs" is not a task. "Add a worked example to docs/workflow.md showing a feature request traced through all six skills" is a task.

---

## Workflow router

Use this table to decide which skill to invoke before doing any work:

| Situation | Skill |
|---|---|
| Need to scope new work for this repo | `/breakdown` |
| Writing a task document | `/write-task` |
| Verifying task docs are self-contained | `/verify-task` |
| Generating IMPLEMENTATION.md from task docs | `/create-tracker` |
| Implementing a scoped task | `/implement` |
| Writing a PR description | `/create-pr` |

**Never begin implementation without a task document.** If you don't have one, use `/write-task` first.

---

## Conventions

### Writing style
- Use plain sentences. No em dashes. No marketing language.
- Prose in docs should read like it was written by a senior engineer explaining a real decision, not a product pitch.
- Code examples should be runnable — no pseudocode unless clearly labeled.

### File naming
- All docs: `kebab-case.md`
- All skill files: `kebab-case.md` matching the skill command name

### Commit format
```
type(scope): short description

Types: feat, fix, docs, chore, refactor
Scope: the file or directory affected (e.g. docs, skills, templates, readme)
```

---

## Do not

- Do not add dependencies — this repo is documentation and skill definitions only. No package.json, no build step.
- Do not modify skill files without updating both `skills/` and `~/.claude/commands/` — they must stay in sync.
- Do not write placeholder content — every file either contains real content or does not exist.
- Do not create architecture or design docs without a task document approving the scope first.
- Do not commit directly to main — every change is a PR.
