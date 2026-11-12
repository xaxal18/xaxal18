---
name: implementation-plan
description: Use this skill before writing code for any non-trivial change. Triggers include "let's implement X", "build the Y feature", "add Z to the project", or any feature/refactor request larger than a one-line fix. In Russian — "давай реализуем", "напиши фичу", "добавь функцию". The plan precedes coding so the user can correct the approach before files change.
---

# Implementation Plan

## Purpose

Before writing code, produce a plan the user can correct. Catches
misunderstandings while they're cheap.

## When to use

- Any change touching 2+ files
- Any change to public APIs, schemas, contracts
- Any feature larger than a single function
- Any time the right approach isn't obvious

## When not to use

- One-line bugfix or typo
- Pure cosmetic edit
- The user explicitly says "just do it"

## Process

### 1. Restate the goal

One sentence. What does success look like? What's the user-visible
behavior change?

If unclear — ask one blocking question. Don't plan on a guess.

### 2. List files to change

- **New** — files to create, with one-line purpose
- **Modify** — files to edit, with what changes
- **Delete** — files to remove (rare, double-check)

Read each file before listing it. Don't invent paths.

### 3. Step-by-step plan

Each step: smallest unit that's independently reviewable.

Bad: "Implement the feature."
Good:
1. Add migration for new column
2. Update model to expose the column
3. Add validation in API handler
4. Wire up UI form field
5. Add unit test for the validation
6. Add E2E test for the flow

Number them. Mark order dependencies.

### 4. Risks

What could break that's not obvious?
- Data integrity (migrations on production)
- Compatibility (clients on old API)
- Performance (new query on a hot path)
- Security (new attack surface)
- UX (changes a familiar flow)

For each: severity + how to mitigate.

### 5. Tests to run

After implementation:
- Unit tests: which files
- Integration tests: which flows
- Manual smoke: what to click / curl
- Typecheck / lint / build

### 6. Rollback plan

If this ships and breaks production, how do we get back?
- Pure code change → revert the commit
- Schema change → reverse migration ready?
- Data migration → idempotent? Recoverable?
- Third-party state change → can we undo it?

### 7. Acceptance criteria

A checklist:
- [ ] Behavior X works as described
- [ ] No regression in flow Y
- [ ] Tests pass
- [ ] Lint/typecheck/build green
- [ ] Docs updated (if user-facing)

## Output template

```markdown
# Plan: <feature>

## Goal
...

## Files
**New:**
- `path/a.ts` — purpose
**Modify:**
- `path/b.ts` — change X
**Delete:**
- `path/c.ts` — why

## Steps
1. ...

## Risks
| Risk | Severity | Mitigation |
|---|---|---|

## Tests
- Unit: ...
- Integration: ...
- Manual: ...

## Rollback
- ...

## Acceptance
- [ ] ...
```

## Anti-patterns

- Skipping the plan because "it's simple"
- Steps that are too big (one step = three subsystems = no)
- Skipping rollback ("it'll be fine")
- Inventing function signatures — read actual code first
- Asking 5 questions before drafting anything
