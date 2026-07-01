---
inclusion: fileMatch
fileMatchPattern: '**/lib/apivex/**,**/lib/rentcast/**,**/services/scraper/**,**/lib/data-sources/**'
---

# Data Sources & API Inventory

## Active Data Sources
- **APIVex (Zillow)** — PRIMARY source for property details, rent estimates, sold comps, and for-sale listing detection. Endpoint: `api.apivex.com/zillow/search_address?address=...`. Header: `x-apivex-key`. Good urban coverage, weak on manufactured/rural (returns null -> derived estimate kicks in).
- **APIVex (Realtor)** — SECONDARY for property details + for-sale listing detection. Returns beds, baths, sqft, lot, year built, photos (tagged by room!), property condition, MLS source, mortgage rates, environmental risk, schools. Endpoint: `api.apivex.com/realtor/property/details?address=...`. Richer detail than Zillow for some properties.
- **APIVex (AirDNA)** — Short-term rental analytics. Returns estimated annual revenue, ADR, occupancy, comparable Airbnb/Vrbo listings, seasonality, amenity benchmarks. Endpoint: `api.apivex.com/airdna/rentalizer?address=...&bedrooms=X&bathrooms=Y`.

## Removed Data Sources
- **RentCast** — REMOVED from all user-facing code. Was previously the fallback for rent estimates, sold comps, active listings, and property details. Replaced by APIVex + derived estimates.

## APIVex Rate Limiting (CRITICAL)
- **Shared limit: 2 requests/second across ALL APIVex endpoints** (Realtor, Zillow, AirDNA all share the same quota).
- A global rate limiter (`lib/apivex/rateLimiter.ts`) enforces this at the process level. Both realtorClient and zillowClient use `rateLimitedFetch` which tracks call timestamps and waits if needed.
- On 403/429 responses, the client automatically retries after 1.1 seconds (up to 2 retries).
- When writing scripts or tests that make multiple APIVex calls, always go through `rateLimitedFetch` or add manual delays of **3+ seconds** between calls.

## APIVex Response Shapes (CRITICAL — don't guess these)
- **Realtor `/property/details`**: `{ data: { description: {...}, location: {...}, photos: [...], ... } }` or property at top level
- **Realtor `/search/forsold`**: `{ count, properties: [...], total }` — NOT `data.results`
- **Realtor `/search/forsale`**: `{ count, properties: [...], total }` — same as forsold
- **Zillow `/search_address`**: Property object directly in response (beds, baths, zestimate, rentZestimate as top-level fields)
- **Zillow `/search`**: `{ results: [...] }` — array of property objects
- **Google Maps/Places** — Address autocomplete, geocoding.
- **OpenAI** — AI report generation, repair cost analysis, strategy narratives.

## Deprecated / Do NOT Use
- **ATTOM** — DEPRECATED. Annual contract, too expensive, key expired. Do NOT call this API.
- **Cotality (CoreLogic)** — DEPRECATED. Annual contract, too expensive. Do NOT call this API.
- **Bridge Data Output** — REMOVED. Static API key expired and can't be refreshed. Zillow sold comps via APIVex provide equivalent data.

## Rules
- **APIVex (Zillow) is the PRIMARY source** for property details and rent estimates.
- **APIVex (Realtor) is the PRIMARY source for sold comps** — use `/search/forsold` with filters.
- Never suggest using ATTOM or Cotality. They are dead.
- Credits are ONLY deducted after successful completion. Never charge for failures.

## Boundary Validation (CRITICAL — from production incident)

External APIs are untrusted data sources. Fields documented as "string" can arrive as numbers, arrays, booleans, null, or objects.

**Mandatory pattern when consuming external API data:**
1. **Validate at the boundary.** Use `lib/apivex/responseValidator.ts` (`asString`, `asNumber`, `asArray`) when extracting fields from raw API responses.
2. **Normalize once, trust downstream.** Client functions are responsible for returning clean, typed data. Business logic should never re-validate.
3. **No `as any` on external data paths.** If you need `as any` to access a field from an API response, that's a signal you need runtime validation.
4. **Test with edge-case shapes.** When adding new API consumption code, consider: what if this field is `null`? A number? An empty array?
