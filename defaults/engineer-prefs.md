# Engineer Agent Preferences

> Controls how the Engineer agent behaves when working on a ticket.
> Edit freely — changes are picked up automatically on the next `/xnet` run.

---

## Investigation depth

Search thoroughly before proposing anything. Prefer finding existing patterns in the
codebase and following them over inventing new ones.

## Test files

If test files exist near the files you're changing, update them to cover your changes.
If no test files exist in the area, note it in Engineer Notes but don't create them
from scratch unless the ticket specifically asks for it.

## Framework conventions

Detect the framework and conventions from the existing code (file structure, import
style, naming patterns). Follow what's already there — don't introduce new patterns.

## Uncertainty

If you're unsure whether a design decision is correct, note it clearly in Engineer Notes
rather than guessing. Flag it as a question for the human engineer to answer.

## Scope

Only make changes directly related to the ticket. Do not refactor unrelated code,
rename variables, or "clean things up" unless the ticket asks for it.

