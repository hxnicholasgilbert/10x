# Phase 7: Handle PR Review Feedback

Triggered when the user says they want to handle review feedback, or invokes `/xnet review <TICKET_ID>`.

---

## Step 1 — Load workspace

Check that `~/xnet/tickets/<TICKET_ID>/WORKSPACE.md` exists. If it does not, halt:

```
⚠️  No workspace found for <TICKET_ID>.
   Run /xnet <TICKET_ID> first to set up the workspace.
```

Read WORKSPACE.md and extract:
- The **Repos** table → absolute paths and repo names
- The **Pull Requests** table → one PR URL per repo

For any repo missing a PR URL in WORKSPACE.md, attempt to find it:
```bash
gh pr list --head <TICKET_ID> --repo <owner>/<repo> --json url --jq '.[0].url'
```

If no PR is found for a repo, skip it and note it in the output.

Output:
```
📂 Workspace: ~/xnet/tickets/<TICKET_ID>/
   Repos with PRs: <N>
   - <repo-name>: <PR_URL>
   - <repo-name>: <PR_URL>
```

---

## Step 2 — Fetch review feedback

**Repeat for each repo with a PR.** For each, fetch all review comments:

```bash
# Inline code comments (line-specific)
gh api repos/<owner>/<repo>/pulls/<PR_NUMBER>/comments

# General review comments and approvals
gh pr view <PR_URL> --json reviews,comments
```

Output:
```
📬 Fetching review feedback for <TICKET_ID>...
   <repo-name>: <N> comments
   <repo-name>: <N> comments
   Total: <N> comments across <N> repos.
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

Read the relevant file before making changes. If the fix appears to already be present, note it — it may have been applied since the comment was written.

For each must-fix item that needs a change, make the code changes using Edit/Write with absolute paths.

After each logical group of changes, commit:
```bash
git -C /absolute/path/to/repo add <files>
git -C /absolute/path/to/repo commit -m "[<TICKET_ID>] Address PR review feedback: <brief description>"
```

---

## Step 5 — Push and reply

Push the updated branch (only if changes were committed):
```bash
git -C /absolute/path/to/repo push origin <TICKET_ID>
```

Reply to each resolved comment thread. Use the `in_reply_to` form — **not** the `/replies` endpoint (which returns 404). Requires `commit_id` (HEAD of the branch after push), `path`, and `line` from the original comment:
```bash
gh api repos/<owner>/<repo>/pulls/<PR_NUMBER>/comments \
  --method POST \
  --field in_reply_to=<comment_id> \
  --field commit_id=<full_sha_of_head_commit> \
  --field path=<file_path> \
  --field line=<original_line_number> \
  --field body="<reply — brief, specific: what was changed and where, or 'already addressed' if no change needed>"
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
