---
name: test-and-debug
description: Use this skill when the user reports a bug, unexpected behavior, failing test, or production incident. Triggers include "X is broken", "this doesn't work", "I'm getting error Y", "test Z is failing", "why does this happen". In Russian — "не работает", "ошибка", "тест падает", "почему так". Always reproduce and form hypotheses before proposing a fix — never patch on guess.
---

# Test and Debug

## Purpose

Diagnose bugs systematically. Find the root cause, not the symptom.
Fix at the right layer. Prevent regression.

## When to use

- Reported bug, error, exception, crash
- Test failing unexpectedly
- "Worked yesterday, doesn't work today"
- Production incident — first calm down, then this skill

## When not to use

- New feature work — use `implementation-plan`
- Performance issue with no error — use `performance-review`
- Architecture concern — use `architecture-review`

## Process — strict order

### 1. Reproduce

Don't fix what you can't reproduce.

- Smallest input that triggers the bug
- Exact command, exact data, exact environment
- If you can't reproduce locally — say so, work from logs

If intermittent: run 5-10 times, note frequency.

### 2. Inspect

- **Error message + stack trace** — start at the deepest project
  frame, not the top
- **Logs around the time of error** — 30 seconds before/after
- **Recent commits** — `git log --oneline -20` and
  `git log -p -- <file>`
- **Dependency versions** — anything updated?
- **Environment differences** — local vs prod env vars

### 3. Form hypotheses (3-5)

Each hypothesis: specific cause + how to verify (concrete check, not
"let's see") + cheapest verification first.

```markdown
## Hypotheses

H1: Migration didn't run on prod, column missing.
  Check: `\d table_name` on prod DB.

H2: Cache stale, returns old data.
  Check: clear cache, retry; check key TTLs in code.

H3: Race condition in async handler.
  Check: instrument with timestamps, look for overlap.
```

### 4. Verify — one at a time

Run the cheapest check first. **Don't propose fixes between
hypotheses.** Eliminate or confirm each, in order.

After each: confirmed / rejected / needs more data.

### 5. Root cause

Once a hypothesis is confirmed — explain *why* this caused the
symptom. "Migration missing" is the cause, "500 error" is the symptom.
The fix targets the cause.

### 6. Minimal fix

Smallest change that fixes the root cause. Resist:
- "While I'm here let me also..."
- "We should refactor X" (separate PR)
- "Add try/except everywhere" (hides the problem)

### 7. Regression test

If non-trivial — write a test that fails before the fix and passes
after. Lives in the test suite (not a debug script).

Skip only if: trivial typo, env-only fix, infra-only fix.

### 8. Verify the fix end-to-end

- Run the failing test — passes?
- Run broader test suite — nothing else broke?
- Reproduce the original bug manually — gone?

### 9. Explain in the commit / PR

Root cause goes in the commit message. Future you will thank you.

```
fix(api): handle null user in middleware

Root cause: middleware assumed user was always set after auth, but
unauthenticated routes still passed through it. Returns early now if
user is null instead of crashing on .id access.
```

## Anti-patterns

- Patching the symptom (`if (x === null) x = {}`) without
  understanding why x is null
- `try/except: pass` to "make it work"
- "I'll just upgrade dependencies, that usually helps"
- Multiple changes per fix — can't tell which one helped
- Skipping the regression test because the fix "feels obvious"
- Fixing the test instead of the bug

## When you genuinely can't reproduce

- Document what you tried
- Add structured logging at the suspected site
- Ship the logging, wait for next occurrence
- Don't pretend you fixed it
