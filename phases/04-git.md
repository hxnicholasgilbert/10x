# Phase 4: Set Up Each Repo

Output: `⚙️  Setting up repos...`

**Repeat for every confirmed repo. Do not proceed to Phase 5 until ALL repos are processed.**

For each repo, choose strategy based on what the user provided:

---

## Worktree Strategy (local path provided)

```bash
LOCAL_PATH="<user-provided local path>"
TICKET_BRANCH="<TICKET_ID>"
WORKTREE_PATH="$HOME/xnet/tickets/<TICKET_ID>/<repo-name>"

cd "$LOCAL_PATH"

# 1. Fetch latest remote state
git fetch origin

# 2. Determine base branch: prefer staging, fall back to main, then master
BASE_BRANCH=""
git show-ref --verify --quiet refs/remotes/origin/staging && BASE_BRANCH="origin/staging"
[ -z "$BASE_BRANCH" ] && git show-ref --verify --quiet refs/remotes/origin/main && BASE_BRANCH="origin/main"
[ -z "$BASE_BRANCH" ] && git show-ref --verify --quiet refs/remotes/origin/master && BASE_BRANCH="origin/master"

if [ -z "$BASE_BRANCH" ]; then
  echo "⚠️  Could not detect base branch. Ask user which branch to use."
  # Wait for user input, set BASE_BRANCH, continue
fi

# 3. Add worktree (use existing branch if it already exists)
if git show-ref --verify --quiet "refs/heads/$TICKET_BRANCH"; then
  git worktree add "$WORKTREE_PATH" "$TICKET_BRANCH"
  echo "✓ <repo-name>: worktree on existing branch $TICKET_BRANCH"
else
  git worktree add -b "$TICKET_BRANCH" "$WORKTREE_PATH" "$BASE_BRANCH"
  echo "✓ <repo-name>: worktree on new branch $TICKET_BRANCH (from $BASE_BRANCH)"
fi
```

Report success or error before moving to the next repo.

**Then register in `~/xnet/config/repos.json`** — auto-derive description from `package.json` → `description`, or first meaningful line of `README.md`:

```json
{
  "name": "<repo-name>",
  "local_path": "<LOCAL_PATH>",
  "clone_url": null,
  "description": "<auto-derived>",
  "strategy": "worktree",
  "last_used": "<today>",
  "tickets": ["<TICKET_ID>"]
}
```

If repo already exists in the file: update `last_used`, append `TICKET_ID` to `tickets` (no duplicates).

---

## Clone Strategy (URL provided)

```bash
WORKTREE_PATH="$HOME/xnet/tickets/<TICKET_ID>/<repo-name>"

# 1. Clone into ticket workspace
git clone <url> "$WORKTREE_PATH"
cd "$WORKTREE_PATH"

# 2. Determine base branch
BASE_BRANCH=""
git show-ref --verify --quiet refs/remotes/origin/staging && BASE_BRANCH="staging"
[ -z "$BASE_BRANCH" ] && git show-ref --verify --quiet refs/remotes/origin/main && BASE_BRANCH="main"
[ -z "$BASE_BRANCH" ] && git show-ref --verify --quiet refs/remotes/origin/master && BASE_BRANCH="master"

if [ -z "$BASE_BRANCH" ]; then
  echo "⚠️  Could not detect base branch. Ask user which branch to use."
  # Wait for user input, set BASE_BRANCH, continue
fi

# 3. Checkout base, pull latest, create ticket branch
git checkout "$BASE_BRANCH"
git pull origin "$BASE_BRANCH"
git checkout -b "<TICKET_ID>"
echo "✓ <repo-name>: cloned, on branch <TICKET_ID> (from $BASE_BRANCH)"
```

Report success or error before moving to the next repo.

**Then register in `~/xnet/config/repos.json`:**

```json
{
  "name": "<repo-name>",
  "local_path": null,
  "clone_url": "<url>",
  "description": "<auto-derived>",
  "strategy": "clone",
  "last_used": "<today>",
  "tickets": ["<TICKET_ID>"]
}
```

If repo already exists: update `last_used`, append `TICKET_ID` to `tickets`.

---

**After all repos are processed, proceed to Phase 5.**
