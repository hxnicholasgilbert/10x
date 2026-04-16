# Phase 9: Cleanup Workspace

Triggered by `/xnet cleanup <TICKET_ID>`, or offered at the end of a completed ticket flow.

---

## Step 1 — Confirm

Read `~/xnet/tickets/<TICKET_ID>/WORKSPACE.md` to get repo paths and strategy (worktree or clone).

Output and wait for confirmation:

```
🧹 Cleanup <TICKET_ID>?

   This will remove:
   - ~/xnet/tickets/<TICKET_ID>/ (workspace files + repos)
   - git worktree for each repo (if worktree strategy)

   Branch <TICKET_ID> will be left intact on the remote.

   Proceed? y/n
```

---

## Step 2 — Remove worktrees

For each repo using the **worktree** strategy:

```bash
git -C <local_repo_path> worktree remove /Users/<username>/xnet/tickets/<TICKET_ID>/<repo-name> --force
```

For repos using the **clone** strategy, the directory removal in Step 3 is sufficient.

---

## Step 3 — Remove workspace directory

```bash
rm -rf ~/xnet/tickets/<TICKET_ID>/
```

---

## Step 4 — Update registry

Remove `<TICKET_ID>` from the `tickets` array for each affected repo in `~/xnet/config/repos.json`. Do not remove the repo entry itself.

---

## Step 5 — Output

```
✅ Cleaned up <TICKET_ID>

   Removed: ~/xnet/tickets/<TICKET_ID>/
   Worktrees removed: <N>
   Branch <TICKET_ID> still exists on remote — delete manually if done.
```

---

## Error Handling

| Situation | Action |
|-----------|--------|
| WORKSPACE.md missing | Ask user to confirm paths manually before proceeding |
| Worktree remove fails (dirty) | Show error, ask whether to force or skip |
| Directory already gone | Note it and continue |
