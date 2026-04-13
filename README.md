# xnet

> Turn a Jira ticket ID into a fully scaffolded engineering workspace in one command.

## Usage

```
/xnet PD-3494
```

That's it. xnet reads the ticket, confirms which repos are involved, sets up branches, and hands off to an Engineer agent that investigates the codebase, makes changes, and writes a PR description — ready to push.

---

## Prerequisites

| Requirement | Notes |
|-------------|-------|
| [Claude Code](https://claude.ai/code) | CLI or desktop app |
| Jira MCP | Connect via `/mcp` → Atlassian. Required — xnet can't read tickets without it. |
| `git` | Must be on your PATH |
| `gh` (GitHub CLI) | Required only for pushing PRs. Install: `brew install gh` |
| Figma MCP *(optional)* | Connect via `/mcp` → Figma. Enables design context in ticket-breakdown.md for design tickets. |

---

## Install

1. **Copy the skill into your Claude Code skills directory:**

   ```bash
   cp -r xnet ~/.claude/skills/xnet
   ```

2. **That's all.** On your first `/xnet` run, the skill bootstraps `~/xnet/config/` from the bundled defaults automatically.

---

## What happens when you run it

```
/xnet PD-3494
```

| Phase | What it does |
|-------|-------------|
| 📋 Read ticket | Connects to Jira, reads the full ticket and acceptance criteria |
| 🎨 Design context *(if Figma link found)* | Asks if you want to pull frame/component specs from Figma |
| 🗂️ Confirm repos | Surfaces context clues from the ticket + known repos from past runs. You confirm which repos are involved. |
| 🗂️ Create workspace | Writes `~/xnet/tickets/PD-3494/ticket.md` and `ticket-breakdown.md` — a plain-English brief with diagrams, scope table, and acceptance criteria |
| ⚙️ Set up repos | Finds each repo locally (git worktree) or clones it. Creates a branch named `PD-3494` from `origin/staging`. |
| 🤖 Engineer agent | Investigates the codebase, makes changes, commits atomically, writes `proposed-solution.md` |
| 🚀 Push PR *(optional)* | Asks if you want to open a PR. Uses `proposed-solution.md` as the description. |

---

## Workspace structure

After a run, your workspace looks like:

```
~/xnet/
├── config/                      ← your personal config (edit freely)
│   ├── pr-template.md
│   ├── engineer-prefs.md
│   ├── commit-style.md
│   └── repos.json               ← registry of repos xnet has used
└── tickets/
    └── PD-3494/
        ├── ticket.md            ← raw Jira content
        ├── ticket-breakdown.md  ← plain-English brief + Engineer progress
        ├── WORKSPACE.md         ← absolute paths for the Engineer agent
        ├── proposed-solution.md ← PR description (written when Engineer is done)
        └── my-repo/             ← git worktree or clone on branch PD-3494
```

---

## Customising behaviour

Edit the files in `~/xnet/config/` — changes are picked up on the next `/xnet` run.

| File | Controls |
|------|----------|
| `engineer-prefs.md` | How the Engineer agent investigates and implements (depth, test coverage, uncertainty handling) |
| `pr-template.md` | Format of `proposed-solution.md` and the PR description |
| `commit-style.md` | Commit message format, atomicity rules |

**Personal additions to the Engineer agent:** append your own instructions to the bottom of `engineer-prefs.md`. Note that slash commands (e.g. `/tokenwise`) don't work inside agent prompts — describe the behaviour in plain text instead (e.g. `Be extremely concise. No preamble.`).

---

## Repo registry

xnet remembers repos you've used in `~/xnet/config/repos.json`. On future runs it will suggest known repos based on ticket context clues — you don't need to type paths twice.

## Tip: skip tool approval prompts

By default Claude Code asks you to approve each tool call. To run xnet without interruption, type `/acceptedits` at the start of your session — this auto-approves tool calls for the session.

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| `⚠️ Jira MCP is not authenticated` | Run `/mcp` in a new Claude Code session, connect Atlassian, then retry |
| `⚠️ Figma MCP unavailable` | Run `/mcp`, connect Figma, then retry — or just say `n` to skip design context |
| Worktree path already exists | xnet will ask: skip or remove the existing worktree |
| Base branch not detected | xnet will ask which branch to use |
| `gh: command not found` | `brew install gh`, then `gh auth login` |
