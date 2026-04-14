# Phase 7: Handle PR Review Feedback

Triggered when the user says they want to handle review feedback, or invokes `/xnet review <TICKET_ID>`.

---

## Step 1 — Load workspace

Read `~/xnet/tickets/<TICKET_ID>/WORKSPACE.md` to get repo paths and PR URL.

If PR URL is not in WORKSPACE.md, fetch it:
```bash
gh pr list --head <TICKET_ID> --json url --jq '.[0].url'
```

If no PR found, tell the user and halt.

---

## Step 2 — Fetch review feedback

For each repo, fetch all PR review comments:

```bash
# Review threads (inline code comments)
gh api repos/<owner>/<repo>/pulls/<PR_NUMBER>/comments

# General review comments and approvals
gh pr view <PR_URL> --json reviews,comments
```

Output:
```
📬 Fetching review feedback for <TICKET_ID>...
   <N> review comments found across <N> repos.
```

---

## Step 3 — Parse and group feedback

Read all comments and group into:

| Category | Description |
|----------|-------------|
| **Must fix** | Explicit change requests, blocking reviews |
| **Suggestions** | Non-blocking improvements or alternatives |
| **Questions** | Reviewer asking for clarification |
| **Praise** | Positive comments — no action needed |

Output a summary:
```
📋 Review summary:
   ❗ Must fix: <N>
   💡 Suggestions: <N>
   ❓ Questions: <N>
```

Then list each must-fix item clearly so the engineer can see what's needed before the agent starts.

Ask:
```
Shall I apply the must-fix changes and respond to comments?
  [y] Yes — apply changes, commit, push, reply to comments
  [n] No — I'll handle manually
```

Wait for confirmation.

---

## Step 4 — Apply changes

For each must-fix item, make the necessary code changes in the workspace repo using Edit/Write with absolute paths.

After each logical group of changes, commit:
```bash
git -C /absolute/path/to/repo add <files>
git -C /absolute/path/to/repo commit -m "[<TICKET_ID>] Address PR review feedback: <brief description>"
```

---

## Step 5 — Push and reply

Push the updated branch:
```bash
git -C /absolute/path/to/repo push origin <TICKET_ID>
```

Reply to each resolved comment thread via the GitHub API:
```bash
gh api repos/<owner>/<repo>/pulls/comments/<comment_id>/replies \
  --method POST \
  --field body="<reply — brief, specific: what was changed and where>"
```

For questions the engineer needs to answer themselves, flag them clearly in chat rather than replying on their behalf.

---

## Step 6 — Update proposed-solution.md

If the approach changed significantly due to review feedback, update `~/xnet/tickets/<TICKET_ID>/proposed-solution.md` to reflect the final implementation.

---

## Step 7 — Summary

Output:
```
✅ Review feedback addressed for <TICKET_ID>

   Applied: <N> must-fix changes
   Replied to: <N> comments
   Pushed to branch: <TICKET_ID>

   Open questions for you to answer manually:
   - <question 1>
   - <question 2>

   PR: <PR_URL>
```
