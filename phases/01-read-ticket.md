# Phase 1: Read the Ticket

Output: `📋 Connecting to Jira...`

## Step 1a — Authenticate

Attempt to call `getAccessibleAtlassianResources`.

- If it **succeeds**: store the returned `cloudId` and site `url`. Output `✅ Jira connected`.
- If it **fails**:

  ```
  ⚠️  Jira MCP is not authenticated.

  Run `/mcp` in a new Claude Code session to connect Atlassian, then restart
  this session and invoke /xnet again.

  Cannot proceed without Jira access.
  ```

  **STOP. Do not continue.**

## Step 1b — Read the ticket

Output: `📥 Reading ticket <TICKET_ID>...`

Call `getJiraIssue` with:
- `cloudId`: from step 1a
- `issueIdOrKey`: the provided ticket ID
- `responseContentFormat`: `"markdown"`

Output: `✅ Ticket loaded: <summary>`

Extract and store for use across all subsequent phases:

| Variable | Source |
|----------|--------|
| `TICKET_ID` | ticket key, e.g. `PD-3494` |
| `JIRA_URL` | site `url` from step 1a, e.g. `https://holidayextras.jira.com` |
| `summary` | ticket title |
| `description` | full description (markdown) |
| `status` | status name |
| `issuetype` | issue type name |
| `assignee` | assignee name or `"Unassigned"` |
| `acceptance_criteria` | explicitly listed ACs, or derived from description |
| `FIGMA_URLS` | all Figma URLs found in description (empty list if none) |

## Step 1c — Detect Figma links (silent)

Scan the ticket `description` for any URLs matching `figma.com`. Collect all matches into `FIGMA_URLS`.

Do not prompt the user here. The Figma question is combined with the repo question in Phase 2.
