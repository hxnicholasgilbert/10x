# Phase 8: Status Overview

Triggered by `/xnet status`.

---

## Step 1 — Scan workspaces

List all ticket directories in `~/xnet/tickets/`. If none exist, output:

```
No active workspaces. Run /xnet <TICKET_ID> to start one.
```

---

## Step 2 — Read each workspace

For each ticket directory, read:
- `WORKSPACE.md` — ticket summary, repos, PR URLs
- `ticket-breakdown.md` — Engineer progress checklist and notes

From these, extract:
- Ticket ID and one-line summary
- Repos involved
- PR URL(s) if any
- Which checklist items are checked off

---

## Step 3 — Output

Print a compact overview. One line per ticket, keep it tight:

```
🗂  xnet — active workspaces

  PD-3494  ✓ PR open     Add name field to Parkcare guest customers (50/50 split test)
  PD-3445  ✓ PR open     Closeout assistant RPC functions — 3 review comments pending
  PD-3501  ⚙ In progress Refactor booking confirmation emails

  3 workspaces  ·  run /xnet cleanup <ID> to remove one
```

**Status icons:**
| Icon | Meaning |
|------|---------|
| ✓ PR open | PR exists (draft or ready) |
| ✓ PR merged | Branch merged — candidate for cleanup |
| ⚙ In progress | Engineer working, no PR yet |
| 📋 Not started | Workspace created, Engineer not run |

Keep each line to one sentence. The summary should tell you what the ticket is about and where it's up to — not repeat the ticket ID or status icon in words.
