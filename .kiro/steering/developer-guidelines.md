---
inclusion: fileMatch
fileMatchPattern: '**/app/api/**,**/lib/db/**,**/lib/validation/**'
---
# Developer Guidelines

## Route Configuration

All API routes MUST be configured for dynamic rendering:

```typescript
export const dynamic = 'force-dynamic';
export const revalidate = 0;
```

Without this: Next.js attempts static generation at build time → ECONNREFUSED on DB, "Dynamic server usage" warnings.

## Database

- Use connection pool from `lib/db.ts` (`getPool()`)
- Never create connections in route handlers
- All DB queries must be in dynamic routes or client-side code

## Security Patterns

- Input validation: Zod schemas with `safeParse()` → 400 on failure
- SQL: Always parameterized queries (`pool.execute('... WHERE x = ?', [value])`)
- Error handling: try/catch → 500 with generic message, `console.error` with details

## Naming Conventions

- Files: camelCase for utilities, PascalCase for components
- Components: PascalCase (`QuizContainer.tsx`)
- Functions: camelCase (`calculateLeadScore`)
- Constants: UPPER_SNAKE_CASE (`MAX_PROPERTIES`)
- Types: PascalCase (`QuizResponse`)
