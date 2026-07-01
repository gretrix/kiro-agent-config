---
inclusion: fileMatch
fileMatchPattern: '**/lib/apivex/**,**/lib/rentcast/**,scripts/test-*,scripts/spike-*'
---

# External API Testing

## Principle
When integrating a new external API, always make a single raw request and inspect the actual response body BEFORE writing any parsing code. APIs lie about their response shapes — the docs may show placeholder `{ "data": {} }` but the real response uses `{ "properties": [...] }` or returns `{"error":"..."}` with HTTP 200.

## Mandatory Sequence (DO NOT SKIP)

**Before writing ANY client code for a new API:**
1. Make ONE raw fetch call in a throwaway script
2. Print the full response (status, headers, body)
3. Note the actual response shape, field names, and edge cases
4. THEN write the client code based on what you observed

**Before writing tests that make multiple API calls:**
1. Confirm the env var loads correctly (`console.log(key?.slice(0,10))`)
2. Make one successful call first, confirm HTTP 200 + valid body
3. THEN run the multi-call test

This sequence prevents the #1 source of wasted time: building a full module on wrong assumptions, then debugging backwards.

## Rules

1. **Test the raw response first.** Before writing a client module, make one `fetch()` call in a standalone script and `console.log(JSON.stringify(data, null, 2))`. Inspect the actual keys, nesting, and edge cases.

2. **Check for error-in-success responses.** Some APIs (APIVex, others) return HTTP 200 with `{"error":"Invalid address"}` in the body. Always check `if (data.error)` after parsing JSON, even when `res.ok` is true.

3. **Respect rate limits in test scripts.** Add delays between calls. For APIVex: minimum 3 seconds between calls in test scripts (the 2/sec limit is stricter than expected at the AWS API Gateway level).

4. **Use `.env.local` explicitly.** Next.js projects use `.env.local`, not `.env`. In test scripts always use `dotenv.config({ path: '.env.local' })`, never `import 'dotenv/config'`.

5. **Module singletons don't survive webpack.** Never use module-level `let` variables as singletons in code that might be dynamically imported in Next.js production builds. Use `globalThis` for process-level shared state.

6. **Normalize external data at the boundary.** Don't pass raw API field values (like `"single_family"`) into internal functions that expect normalized values (like `"Single Family"`). Write a normalizer at the client layer.
