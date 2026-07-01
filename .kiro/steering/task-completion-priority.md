---
inclusion: manual
---
# Task Completion Priority

## Principle
Finish what you started before moving to new requests. Don't leave work half-done.

If the user adds something mid-task, acknowledge it and finish the current work first — unless they clearly want you to switch. Use judgment, not keyword matching.

Before reporting completion: verify the work is actually complete across its full scope. If you changed something in one place that should be consistent across others, apply it everywhere — not just the one spot you happened to touch.

## Document Before Doing
When the user gives you a task (or multiple tasks in one message), **write them to `.snapshots/active-tasks.md` FIRST** — before starting implementation. This ensures:
- Tasks survive if you get interrupted or context is compacted
- Other sessions can see what this session is working on (prevents conflicts)
- The user can verify you understood the full scope before you act

Format: short title + one-line description + status (TODO / IN PROGRESS / BLOCKED / DONE).

When the user gives you 3 things and you only finish 2, the third is still in `active-tasks.md` for next time. When you complete a task, mark it DONE in the same file.

Exception: trivial one-liner fixes (typo, single-line config change) don't need documentation.

## Atomic Edits
Every individual file edit must leave the file in a compilable state. Never make a partial edit (e.g., add new declarations without removing the old ones) and move on to the next turn. If a change requires multiple str_replace calls to be coherent, do them ALL in the same turn or rewrite the whole file. A broken intermediate state that survives across turns compounds into harder bugs.
