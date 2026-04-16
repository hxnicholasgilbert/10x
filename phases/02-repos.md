# Phase 2: Identify Related Repos

## Step 2a — Check the repo registry

Check whether `~/xnet/config/repos.json` exists and read it. This file tracks repos xnet has seen before with their clone URLs, local paths, and descriptions.

If relevant entries exist, include them as pre-filled suggestions so the engineer can confirm rather than re-enter paths.

## Step 2b — GitHub auto-search (optional)

If `gh` is available, offer to scan GitHub for repos related to this ticket before asking the user to confirm manually:

```
Auto-search GitHub for repos linked to <TICKET_ID>?
Scans for PRs and branches matching the ticket ID across the org. May take a moment.
y/n (enter to skip)
```

If yes, run:

```bash
# PRs mentioning the ticket ID
gh search prs "<TICKET_ID>" --json repository,title,url --limit 10

# Branches named after the ticket in known repos (from registry)
# For each repo in repos.json:
gh api repos/<owner>/<repo>/branches --jq '[.[] | select(.name | contains("<TICKET_ID>")) | {branch: .name, repo: "<repo>"}]'
```

Collect any repos surfaced by either command into a candidate list. These are folded into the prompt in Step 2c as **GitHub matches** — the user confirms or removes them alongside registry entries.

If `gh` is not available or user skips, continue to Step 2c with registry entries only.

---

## Step 2c — Single prompt: repos + Figma

Send **one** prompt combining repo confirmation and (if applicable) the Figma question. Do not split these into separate interactions.

```
## Ticket: <TICKET_ID> — <summary>

**Type:** <issuetype> | **Status:** <status> | **Assignee:** <assignee>

### Context clues
- Platform/product area: [extracted from ticket]
- Teams mentioned: [extracted from ticket]
- Technology hints: [e.g. feature flag, A/B test, API, frontend]
- Feature names: [extracted from ticket]
- Figma/design refs: [extracted from ticket, or "none"]

### Known repos (from previous xnet sessions)
[Include only if registry has relevant entries:]

| Repo | Path / URL | About |
|------|-----------|-------|
| parkcare | /Users/nick/code/parkcare | Parkcare platform — booking and details flow |

### GitHub matches (auto-found)
[Include only if Step 2b surfaced results:]

| Repo | Branch / PR |
|------|------------|
| tripapplite | branch: PD-3494 |
| llm-fn-proxy | PR #854 — Add closeout RPC functions |

Confirm which of these to include — add ✓ or ✗, or leave blank to skip.

### Additional repos
| Repo name | Local path or clone URL |
|-----------|------------------------|
| (add any not listed above) | /path/to/repo  OR  git@github.com:org/repo.git |

[Include only if FIGMA_URLS is non-empty:]
### Figma
This ticket links to Figma: <list URLs>
Pull design context into ticket-breakdown.md? y/n
```

**Wait for the user's response.** Parse out:
- Final confirmed list of repo names and their paths/URLs
- Whether to fetch Figma context (`FETCH_FIGMA` = true/false)

## Step 2c — Fetch Figma context (if requested)

If `FETCH_FIGMA` is true, for each URL in `FIGMA_URLS`:
1. Extract the file key from the URL path (segment after `/design/` or `/file/`)
2. Extract `node-id` query parameter if present (convert `1-234` format to `1:234` if needed)
3. Use available Figma MCP tools to fetch design data:
   - If node-id present: fetch that specific node/frame
   - If no node-id: fetch top-level file metadata only
4. Store the result as `FIGMA_CONTEXT` — a concise summary of frame/component names, descriptions, annotated specs

If the Figma MCP is unavailable or the fetch fails:
```
⚠️  Figma MCP unavailable. Skipping design context.
   Tip: run /mcp in a new session to connect Figma, then retry.
```
Set `FIGMA_CONTEXT` to empty and continue.

If `FETCH_FIGMA` is false or `FIGMA_URLS` was empty: set `FIGMA_CONTEXT` to empty.

Continue to Phase 3.
