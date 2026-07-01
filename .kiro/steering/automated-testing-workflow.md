---
inclusion: manual
---
# Testing

## Principle
Code changes must be validated before shipping. Don't assume it works because it compiles — prove it works by running it.

## How to Think About Testing

- If tests exist for the area you changed, run them. If they fail, that's a regression — fix it before shipping.
- If you're fixing a bug, the best proof is a test that would have caught it. Write one.
- If you're adding a feature, validate it actually works in the real environment, not just in theory.
- Build passing ≠ feature working. TypeScript catches type errors. It doesn't catch logic errors, runtime crashes, or broken user flows.

## What "Validated" Means

It means you triggered the actual code path and confirmed the outcome. Not "the build passed." Not "getDiagnostics returned 0 errors." Those are necessary but not sufficient. The feature needs to actually work when a real person uses it.

## Real Validation

"Validated" means you triggered the actual code path in production (or production-equivalent) and OBSERVED the expected outcome yourself. Never say "this will work when X happens later." Instead: make X happen NOW.

- Trigger the cron job manually
- Submit the form yourself
- Call the API endpoint and inspect the response
- Run the scrape in test mode
- Check the Slack channel / database / logs for the expected output

If the only way to validate is to wait for a future event, **create that event now** in a controlled way (test mode, manual trigger, temporary schedule change).

You have access to production logs, the database, shell commands, and API endpoints. USE THEM. Do not ask the user to go check something you could verify yourself. If you need access to something you don't have (like viewing a Slack channel), say so explicitly — don't pretend you validated when you didn't.

The standard: the closest thing to a real user doing the real thing, observed by you with your own tools.
