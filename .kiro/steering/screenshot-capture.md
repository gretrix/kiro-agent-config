---
inclusion: fileMatch
fileMatchPattern: '**/scripts/take-*screenshot*,**/scripts/validate-ios*,**/store-assets/**,**/ios-screenshots/**,**/play-store-screenshots/**'
---

# Screenshot Capture — Lessons Learned

## Known Failure Modes (all have burned time before)

1. **Not logged in.** Every authenticated page renders as the login form if the session isn't established. Always login FIRST, wait for the session cookie to be set, and verify you're on an authenticated page before capturing. The test: does the page show user-specific content (credits, name, dashboard data)?

2. **Orientation popup overlay.** The app shows a "rotate your device" modal on mobile viewports. Suppress it via localStorage BEFORE navigating to each page: `localStorage.setItem('orientation-popup-dismissed', 'true')`. Also dismiss any visible dialogs after navigation.

3. **Wrong dimensions.** Apple App Store requires EXACT pixel sizes — not "close enough":
   - iPhone 6.7": **1284×2778** (viewport 428×926 @ 3x scale)
   - iPhone 6.5": **1242×2688**
   - iPad 12.9": **2048×2732** (viewport 1024×1366 @ 2x scale)
   - Play Store: flexible (min 320px, any reasonable aspect ratio)

4. **Blank/skeleton screenshots.** Pages with async data (maps, dashboards) need adequate wait time. A screenshot under 100KB is almost certainly a blank or skeleton page. Verify file size > 100KB after capture.

## Validation Checklist (run after ANY screenshot capture)

```bash
npx tsx scripts/validate-ios-screenshots.ts --fix
```

This checks dimensions AND file sizes. Use `--fix` to auto-crop screenshots that are a few pixels off.

## Suppression Code (copy into any screenshot script)

```typescript
await page.evaluate(() => {
  localStorage.setItem('whats-new-modal-seen-version', '99.0.0');
  localStorage.setItem('pwa-install-dismissed', 'true');
  localStorage.setItem('onboarding-completed', 'true');
  localStorage.setItem('orientation-popup-dismissed', 'true');
  localStorage.setItem('device-orientation-dismissed', 'true');
  localStorage.setItem('install-prompt-dismissed', 'true');
});
```

## The Existing Capture Script

`scripts/take-all-store-screenshots.ts` handles all of this correctly:
- Logs in with credentials from `.env.local`
- Suppresses all overlays
- Verifies each screenshot > 100KB (retries up to 3x if blank)
- Outputs to `E:/ios-screenshots/{brand}/{device}/`
- Runs validation as post-capture step

Use it. Don't reinvent screenshot capture from scratch.
