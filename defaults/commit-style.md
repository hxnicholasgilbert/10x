# Commit Style

> Controls how the Engineer agent commits changes.
> Edit freely — changes are picked up automatically on the next `/xnet` run.

---

## Format

```
[<TICKET_ID>] <imperative summary under 72 chars>
```

Examples:
- `[PD-3494] Add feature flag for new checkout flow`
- `[PD-3494] Fix null reference in basket reducer`
- `[PD-3494] Remove deprecated payment provider config`

## Rules

- **Imperative mood** — "Add", "Fix", "Remove", not "Added", "Fixes", "Removed"
- **Atomic commits** — one logical unit per commit (one component, one function, one config change). Never bundle unrelated changes
- **Specific messages** — describe what changed and where. Never: "Update files", "WIP", "Fix stuff"
- **No amend, no force** — create new commits only. Never `git commit --amend` or `git push --force`
- **No noise** — don't commit build artifacts, lock file changes unrelated to your work, or editor config
