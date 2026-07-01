---
inclusion: fileMatch
fileMatchPattern: '**/app/sitemap*,**/app/robots*,**/app/**/route.ts'
---

# Force-Dynamic Rule for Host-Detecting Routes

Any Next.js route file that calls `headers()` for host/domain detection MUST export `const dynamic = 'force-dynamic'` to prevent build-time static generation defaulting to fortuneleo.com.

## Why This Matters

Without `export const dynamic = 'force-dynamic'`, the route renders with a null Host header during `next build` and caches the Fortune Leo version permanently. This means white-label domains (dealsidekick.com, chratl.com) will serve Fortune Leo-branded sitemaps, robots.txt, and API responses.

## Rule

If a route file:
1. Imports `headers` from `next/headers`
2. Uses the `Host` header (or any header) to determine domain/branding

Then it MUST include this export near the top of the file (after imports):

```typescript
export const dynamic = 'force-dynamic';
```

## Examples

```typescript
// app/sitemap.ts
import { MetadataRoute } from 'next';
import { headers } from 'next/headers';

export const dynamic = 'force-dynamic'; // Required!

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const headersList = await headers();
  const host = headersList.get('host') || 'fortuneleo.com';
  // ...
}
```

## Common Violations

- `app/sitemap.ts` — generates per-domain sitemap
- `app/robots.ts` — generates per-domain robots.txt
- `app/api/*/route.ts` — any API route that branches on Host header
- `app/opengraph-image.tsx` — OG images that vary by domain
