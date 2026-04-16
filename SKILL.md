---
name: xnet
description: Ticket workspace builder. Given a Jira ticket ID, reads the ticket via Jira MCP, identifies related repos, scaffolds a ~/xnet/tickets/<ID>/ workspace, sets up repos via git worktree or clone, and spawns an Engineer agent to propose an implementation.
---

# xnet — Ticket Workspace Skill

**Triggers:**
- `/xnet <TICKET-ID>` — full workspace build (Phases 1–6)
- `/xnet review <TICKET-ID>` — handle PR review feedback (Phase 7)
- `/xnet status` — overview of all active workspaces (Phase 8)
- `/xnet cleanup <TICKET-ID>` — remove workspace and worktrees (Phase 9)

---

## Step 0: Bootstrap

Before doing anything else, check whether `~/xnet/config/` exists.

**If it does NOT exist** — this is a first run. Initialise the engineer's workspace:

1. Create `~/xnet/config/`
2. Copy each file from `~/.claude/skills/xnet/defaults/` into `~/xnet/config/`:
   - `pr-template.md`
   - `engineer-prefs.md`
   - `commit-style.md`
3. Output:

```
✅ xnet initialised for the first time.

   Default config created at ~/xnet/config/
   Open ~/xnet in your editor to customise:
     - pr-template.md    → PR description format
     - engineer-prefs.md → how the Engineer agent behaves
     - commit-style.md   → commit message format and rules
```

**Whether freshly created or pre-existing**, load the config into working memory:

- Read `~/xnet/config/pr-template.md` → `PR_TEMPLATE`
- Read `~/xnet/config/engineer-prefs.md` → `ENGINEER_PREFS`
- Read `~/xnet/config/commit-style.md` → `COMMIT_STYLE`

These are passed into the relevant phases below.

---

## Phases

Execute each phase in order. Read the linked phase file for full instructions.

| Phase | File | What it does |
|-------|------|-------------|
| 1 | `phases/01-read-ticket.md` | Authenticate Jira MCP, read ticket data |
| 2 | `phases/02-repos.md` | Check registry, surface clues, confirm repos |
| 3 | `phases/03-workspace.md` | Create workspace dir, write ticket.md + ticket-breakdown.md |
| 4 | `phases/04-git.md` | Set up each repo via worktree or clone, update registry |
| 5 | `phases/05-engineer-agent.md` | Write WORKSPACE.md, spawn Engineer agent |
| 6 | `phases/06-pr.md` | Quick checks, push draft PR, parallel agent comments, mark ready |
| 7 | `phases/07-review.md` | Read PR review comments, apply changes, reply — triggered by `/xnet review <TICKET-ID>` |
| 8 | `phases/08-status.md` | Overview of all active workspaces — triggered by `/xnet status` |
| 9 | `phases/09-cleanup.md` | Remove workspace, worktrees, update registry — triggered by `/xnet cleanup <TICKET-ID>` |

---

## Error Handling

| Situation | Action |
|-----------|--------|
| Jira MCP not authenticated | Show `/mcp` instruction, halt |
| Ticket not found | Report error, ask user to verify ticket ID |
| `git worktree add` fails (path exists) | Show error, ask: skip or remove existing worktree? |
| No base branch found (no staging/main/master) | Ask user which branch to use |
| Clone fails | Show error, ask user to verify URL or check network |
| Engineer agent produces no useful output | Relay output, offer to re-run with narrowed scope |
| `~/xnet/config/` file missing after init | Re-copy the missing file from `defaults/` |
