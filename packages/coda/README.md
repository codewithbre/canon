# Coda

The test and deploy agent. Takes merged code and closes the loop: runs NL-driven tests, deploys to the target environment, and verifies the deployment succeeded.

Part of the [Orchestra](../../README.md) monorepo.

## Relationship to Fugue

```
Fugue  →  builds (plan → implement → PR)
Coda   →  closes (test → deploy → verify)
```

Fugue produces PRs. Coda takes those merged PRs to production and confirms the system still works.

## Status

Planning. Scoping in progress.
