---
inclusion: manual
---

# Cross-boundary verification (API, UI, email, jobs)

## Principle

Most defects do not live “inside one function.” They live at **boundaries**: what one layer puts on the wire, what the next layer **names** the same concept, and what **null / missing / empty** means at each hop. **Align meaning across boundaries** before polishing any single file.

## Mindset (judgment, not a script)

1. **Same word, different contracts** — Fields like `success`, `complete`, or `error` often mean different things for “transport OK,” “business OK,” and “payload present.” **Null can be valid** on a 200 response. The UI must not imply a single failure mode when the server meant something narrower (e.g. “saved, analysis skipped”).

2. **Read producer and consumer together** — Before or while changing a route handler, client `fetch`, email template, or worker consumer, open **both** sides in one reasoning pass. Changing only the client is guessing server guarantees; changing only the server is guessing client tolerance.

3. **Shape categories, not only the happy path** — For each boundary, hold at least three mental (or fixture) categories: **success with full payload**, **success with intentionally empty / null payload**, and **failure** (status or body). Categories beat ad hoc “it worked on my machine.”

4. **Smallest proof that matches the risk** — A scripted POST + JSON parse, a tiny Jest case on the handler’s return shape, or a short Playwright step is enough when the risk is “wrong interpretation across layers.” Match proof size to blast radius.

5. **Teaching the repo** — When friction came from a **class** of mistake (boundary mismatch), prefer a **short steering principle** over a one-off hook. See `.kiro/steering/writing-hooks-and-steering.md` for how to write hooks vs steering so the lesson scales.

## When to load

Say **`#cross-boundary-verification`** when work touches **both** an API route (or server action) and the UI, email, or job that consumes it—or when debugging “submitted but UI says error” / “email says X but DB says Y” class issues.

**Related:** `#writing-hooks-and-steering` (how to encode habits in hooks vs rules vs manual docs).
