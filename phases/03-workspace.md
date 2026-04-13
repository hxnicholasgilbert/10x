# Phase 3: Create Workspace

Output: `🗂️  Creating workspace at ~/xnet/tickets/<TICKET_ID>/...`

Create the workspace directory and write all context files **before** any git work — so context exists even if git setup partially fails.

```bash
mkdir -p ~/xnet/tickets/<TICKET_ID>
```

---

## Write `ticket.md`

Write `~/xnet/tickets/<TICKET_ID>/ticket.md`:

```markdown
# <TICKET_ID>: <summary>

**Type:** <issuetype>
**Status:** <status>
**Assignee:** <assignee>

---

<full ticket description markdown>
```

---

## Write `ticket-breakdown.md`

Write `~/xnet/tickets/<TICKET_ID>/ticket-breakdown.md` as a human-friendly brief. Use GitHub Markdown throughout. Include a Mermaid diagram where the ticket describes a user flow, feature toggle, A/B split, or process. Use tables where multiple items need comparing.

```markdown
# Ticket Breakdown: <TICKET_ID>

> **<summary>**

## What This Ticket Is Trying to Achieve

<2–4 sentences in plain English. Goal, why it matters, who it affects. No jargon.>

## Scope

| Item | In Scope | Notes |
|------|----------|-------|
| <e.g. New/guest customers> | ✅ | <any nuance> |
| <e.g. Logged-in customers> | ❌ | <why excluded> |

## Acceptance Criteria

<Extract from ticket — convert to checkboxes. Derive from description if not explicit.>

- [ ] <criterion 1>
- [ ] <criterion 2>
- [ ] <criterion 3>

## Flow Diagram

<Include if the ticket involves a user journey, toggle, or split. Example:>

​```mermaid
flowchart TD
    A[User visits details page] --> B{New or Guest?}
    B -- Yes --> C{In test group?}
    B -- No --> D[Existing experience]
    C -- Control 50% --> D
    C -- Variant 50% --> E[Details page with name field]
​```

## Affected Areas

| Area | What Changes | Risk |
|------|-------------|------|
| <component/page> | <what changes> | Low / Medium / High |

<IF FIGMA_CONTEXT is non-empty, include this section — otherwise omit entirely:>

## Design Context

> Fetched from Figma. Use this as the source of truth for UI requirements.

<FIGMA_CONTEXT — frame names, component names, annotated specs, layout notes>

<END IF>

---

## Engineer Progress

> Updated by the Engineer agent as work progresses.

- [ ] Codebase investigation complete
- [ ] Implementation plan defined
- [ ] Code changes made
- [ ] Changes committed
- [ ] Testing approach documented

## Engineer Notes

> *This section is filled in by the Engineer agent.*

---
*Workspace created: <today's date> · Branch: <TICKET_ID>*
```

---

After writing both files, output:

```
✅ Workspace ready at ~/xnet/tickets/<TICKET_ID>/

📖 Read ticket-breakdown.md for a plain-English overview while the Engineer agent works.
   It will update the progress checklist and add notes as it goes.
```
