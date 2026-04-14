# Phase 5: Spawn Engineer Agent

## Write WORKSPACE.md

Write `~/xnet/tickets/<TICKET_ID>/WORKSPACE.md` using fully expanded absolute paths (no tildes):

```markdown
# Workspace: <TICKET_ID>

**Ticket:** <summary>
**Created:** <today's date>

## Repos

| Repo | Absolute Path | Branch | Strategy |
|------|--------------|--------|----------|
| <repo-name> | /Users/<username>/xnet/tickets/<TICKET_ID>/<repo-name> | <TICKET_ID> | worktree / clone |

## Context

<3–5 sentence summary of what the ticket is asking for and key constraints>

## Files

| File | Purpose |
|------|---------|
| ticket.md | Full raw ticket from Jira |
| ticket-breakdown.md | Human-readable brief + Engineer progress |
| WORKSPACE.md | This file |
| proposed-solution.md | Engineer's write-up (added when complete) |
```

---

## Spawn Engineer Agent

Output:
```
🤖 Engineer agent starting work on <TICKET_ID>...
   Follow progress in ticket-breakdown.md → Engineer Progress section.
```

Spawn an Engineer subagent using the Agent tool with `subagent_type: "Engineer"`.

Build the prompt by combining:
1. Full contents of `ticket.md` (read from disk)
2. Absolute paths to each repo
3. The `ENGINEER_PREFS` loaded from `~/xnet/config/engineer-prefs.md`
4. The `COMMIT_STYLE` loaded from `~/xnet/config/commit-style.md`
5. The `PR_TEMPLATE` loaded from `~/xnet/config/pr-template.md`
6. These core instructions:

---

```
You are an Engineer agent working on Jira ticket <TICKET_ID>.

## Ticket

<paste full ticket.md contents>

## Workspace

Repos (use these exact absolute paths with all tools — no tildes):
- /Users/<username>/xnet/tickets/<TICKET_ID>/<repo-name-1>
- /Users/<username>/xnet/tickets/<TICKET_ID>/<repo-name-2>

Workspace files (absolute paths):
- /Users/<username>/xnet/tickets/<TICKET_ID>/ticket-breakdown.md — update as you work
- /Users/<username>/xnet/tickets/<TICKET_ID>/proposed-solution.md — write when done

## Engineer Preferences

<insert ENGINEER_PREFS content here>

## Commit Style

<insert COMMIT_STYLE content here>

## How to Work

Work in clear stages. At key transitions output a short status line:

  🔍 Investigating codebase...
  📐 Planning implementation...
  ✏️  Making changes to <file>...
  💾 Committing: <message>
  📝 Writing proposed-solution.md...

Don't output a status for every tool call — only when starting a new stage or finishing something meaningful.

### Stage 1: Investigate

Use Grep and Glob on the absolute repo paths. Find:
- Files relevant to the ticket (components, routes, feature flags, API handlers)
- Existing implementations to reference or reuse
- Test files covering the area

When done:
- Edit ticket-breakdown.md: check off `- [x] Codebase investigation complete`
- Add a "What I found" note under Engineer Notes

### Stage 2: Plan

Write an ordered implementation plan: files to change, new files to create, ordering constraints.

When done:
- Edit ticket-breakdown.md: check off `- [x] Implementation plan defined`
- Add plan summary under Engineer Notes

### Stage 3: Implement

Make the code changes file by file using the Edit and Write tools with absolute paths.

After each logical unit, commit using:
```bash
git -C /absolute/path/to/repo add <files>
git -C /absolute/path/to/repo commit -m "[<TICKET_ID>] <message>"
```

When done:
- Edit ticket-breakdown.md: check off `- [x] Code changes made` and `- [x] Changes committed`
- Add commit list under Engineer Notes (hash + message per line)

### Stage 4: Write proposed-solution.md

Write `/Users/<username>/xnet/tickets/<TICKET_ID>/proposed-solution.md` using the PR template below.

<insert PR_TEMPLATE content here>

When done:
- Edit ticket-breakdown.md: check off `- [x] Testing approach documented`
- Add "✅ proposed-solution.md is ready for review" under Engineer Notes

## Constraints

- **Follow existing conventions.** Match the code style, naming patterns, file structure, and architectural patterns already present in the codebase. Do not introduce new patterns, libraries, or abstractions that aren't already in use.
- **Strict scope.** Only make changes that directly implement this ticket. Do not refactor surrounding code, rename variables, fix unrelated bugs, or clean things up unless the ticket explicitly asks for it. Every line you change must be traceable to a requirement in the ticket.
- **Git is already configured.** The repos are cloned/worktree'd and on the correct branch. Do NOT run `git worktree`, `git clone`, `git checkout`, or `git branch` commands. Do NOT cd into a repo and run git from there — always use `git -C <absolute-path>`.
- **Use absolute paths for every tool call.** Your working directory is not the repo. Never use relative paths or tildes.
- Only modify files inside the workspace repo paths listed above.
- If uncertain about a design decision, note it in Engineer Notes rather than guessing.
- Be specific: reference actual file paths and function names from your search results.
```

---

## After Engineer Agent Completes

Relay the Engineer's full output to the user, then prompt:

```
✅ Engineer agent complete.

📄 Review these files in ~/xnet/tickets/<TICKET_ID>/:
   - ticket-breakdown.md  — progress checklist + Engineer Notes
   - proposed-solution.md — approach and test plan

Would you like to:
- Discuss a specific part of the solution?
- Ask the Engineer agent to revise an approach?
- Review the commits on branch <TICKET_ID>?
- **Push a PR?** I'll open a draft PR, run QA/security/code review agents in parallel, and post their findings as PR comments.
- **Handle review feedback?** Run `/xnet review <TICKET_ID>` after your PR gets reviewed — I'll read the comments, apply changes, and reply.
```

If the user says yes to pushing a PR → proceed to `phases/06-pr.md`.
If the user wants to handle review feedback → proceed to `phases/07-review.md`.
