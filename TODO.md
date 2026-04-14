# TODO

## Jira ticket listener

Automatically trigger xnet when a ticket is raised on the team's board/backlog — no manual `/xnet` invocation needed.

**Flow:**
1. Listen for new tickets added to a configured Jira board/backlog
2. Automatically run xnet for that ticket ID
3. Generate a workspace report (ticket breakdown, proposed solution)
4. Post the report as a comment on the Jira ticket
5. Open a draft PR and reference it in the same comment

**Open questions:**
- Trigger mechanism — Jira webhooks? Polling via MCP? Claude Code scheduled agent?
- Which boards/projects to watch — per-engineer config in `~/xnet/config/`?
- Should it run the full Engineer agent automatically, or just phases 1–3 (workspace + breakdown) and wait for human confirmation before coding?
