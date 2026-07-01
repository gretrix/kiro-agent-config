---
inclusion: fileMatch
fileMatchPattern: '**/hooks/**,**/steering/**,**/services/**,**/scripts/**,**/lib/apivex/**,**/lib/rentcast/**'
---
# Reference Doc Legend

Certain task types require reading specific guidance before proceeding. If the current work matches any category below, read the associated files fully before acting.

| Task Type | Files to Read | When It Applies |
|-----------|--------------|-----------------|
| Hook/Steering changes | `.kiro/steering/writing-hooks-and-steering.md` + `.kiro/hooks/CHANGELOG.md` | Any modification to files in `.kiro/hooks/` or `.kiro/steering/` |
| Deploy/Infrastructure | `#infrastructure-safety` | Server changes, deploy scripts, CI/CD, EC2, PM2, Caddy, blue-green swaps |
| API integrations / data sources | `#data-sources` | Working with APIVex, RentCast, Zillow, Realtor, GSCCCA, or adding new data providers |
| Mobile app work (iOS/Android) | `#server-reference` | Capacitor builds, app store submissions, native features, push notifications |
| UI/UX/visual changes | `#human-centered-design` | Layout, styling, animations, component design, accessibility, responsive behavior |
| Slack alert handling | `#slack-alert-processing` | Reading, resolving, or posting to Slack alert channels |
| Background jobs / scrapers | `#background-jobs-reference` | Scheduler crons, Playwright scrapers, PM2 processes, queue workers |
| Cross-boundary work (API ↔ UI ↔ email) | `#cross-boundary-verification` | Changes that span multiple layers where meaning could drift between them |
| Landing pages / marketing | `#landing-page-framework` | Public-facing pages, SEO, lead funnels, conversion flows |
| Architecture decisions | `#decision-log` | Choosing between approaches, adding new dependencies, structural changes |
| External service access | `.kiro/steering/external-api-access.md` | Anything involving Slack, Cloudflare, Play Store, GitHub Actions, or other third-party APIs |
