# PedalScape team guide

Last updated: 2026-06-17

## Product positioning

- PedalScape is a no-login scenic ride launcher for any indoor bike and screen.
- The public promise is: "Any bike. Any screen. Beautiful rides from home."
- It is intentionally local-first: there is no backend database, account system, or social login.
- The core job is simple: browse curated scenic cycling videos, pick a ride, and launch a fullscreen player on a laptop, tablet, or TV.

## Live site, repo, and hosting

- Live site: `https://pedalscape.com/`
- GitHub repo: `https://github.com/shanselman/ScenicSpin`
- Hosting: GitHub Pages from the `main` branch.
- Custom domain: root `CNAME` contains `pedalscape.com`.
- HTTPS is enforced in GitHub Pages settings.
- Note the repo/package still use some legacy `ScenicSpin` / `scenic-ride-catalog` naming while the product branding is PedalScape.

## Architecture

- Static app only: `index.html`, `src/styles.css`, `src/app.js`, JSON data, icons, and generated images.
- Production catalog: `routes/catalog.json`.
- Candidate/review backlog: `routes/candidate-backlog.json`.
- Hidden review mode:
  - Open with `?review=1` or `#review`.
  - Loads candidates only for triage.
  - Lets reviewers mark Promote/Yes, Reject/No, or Defer/Maybe, add notes, and export decisions.
- PWA:
  - `manifest.webmanifest` declares standalone display, theme colors, categories, and app icons.
  - `service-worker.js` caches the app shell, production catalog, candidate backlog, icons, and core assets.

## Data model and curation workflow

- `routes/catalog.json` is the production catalog. Current size: 40 routes.
- `routes/candidate-backlog.json` is not production. Current size: 32 candidates.
- Catalog route fields include:
  - `id`, `title`, `sourceUrl`, `embedUrl`, `sourcePlatform`, `creator`, `location`
  - `durationMinutes`, `difficulty`, `terrain`, `sceneryTags`
  - `videoQuality`, `audio`, `cameraStyle`, `embeddingAllowed`, `license`, `curationNotes`
- Candidate entries add review/curation fields such as `status`, `curationTier`, `promotionReadiness`, `verification`, `productionCatalogId`, and `promotedToCatalogAt`.
- Promotion heuristics:
  - Prefer official creator/public channel sources, not reposts.
  - Confirm embeds work without downloading or rehosting.
  - Verify location, duration, audio, camera stability, quality, and workout pacing.
  - Promote for catalog diversity across geography, duration, difficulty, terrain, season, and audio style.
  - Positive triage signals include about-30-minute duration, no interruptions, high-quality video, and substantial view count, but these do not replace legitimacy, embed, and legal checks.
- Badge/chicklet philosophy:
  - Use compact, useful badges over raw metadata dumps.
  - Keep route cards visually scannable.
  - Strip reviewer-only text like "verify", "before launch", "do not download", and similar internal notes from public card copy.
  - Limit media badges to five.
- Legal/platform stance:
  - Link/embed official public streams only.
  - Do not download, proxy, mirror, or rehost creator video files.
  - Re-check embed availability, source authenticity, licensing, and platform terms before launch because platform settings can change.

## UX principles learned

- Visual-first cards: thumbnails/preview art lead each route card, with title, location, duration, intensity, and a few useful badges.
- Compact hero: the hero highlights a recommended first ride or a "Continue ride" state without overwhelming browsing.
- Sticky search/filter: filters stay close to the top of the experience and persist locally.
- Hidden review mode: backlog triage exists for maintainers but is not mixed into the public catalog.
- Local-first trust footer: explain that there is no account or backend, and that favorites/preferences stay in this browser.
- Useful curation badges: surface high-signal badges like `4K`, `60+ min`, `Climb`, `Mountains`, `Water/Lakes`, `Natural audio`, or `No music`; avoid noisy/internal review labels on production cards.

## Local-first and PWA behavior

- Favorites, recent routes, selected/continue ride, and filters are stored locally in the browser.
- Local data controls:
  - Save/remove favorites.
  - Track up to five recent routes.
  - Continue the last started ride from the hero.
  - Export/download local backup JSON.
  - Copy backup JSON.
  - Import a validated PedalScape backup.
  - Reset local data.
- Local storage/session keys:
  - `scenicRideCatalog.selectedRouteId` in `sessionStorage` and `localStorage`
  - `scenicRideCatalog.favoriteRouteIds`
  - `scenicRideCatalog.recentRouteIds`
  - `scenicRideCatalog.filterPreferences`
  - `PedalScape.reviewDecisions`
- Backup format:
  - `app: "PedalScape"`
  - `schemaVersion: 1`
  - `localData.selectedRouteId`
  - `localData.favoriteRouteIds`
  - `localData.recentRouteIds`
  - `localData.filterPreferences`
- Offline/install behavior:
  - The app shell, catalog JSON, backlog JSON, and icons are cached by the service worker.
  - The app can render the shell/catalog offline after caching.
  - Ride videos and YouTube thumbnails are third-party network resources and still require internet.

## Testing and validation

- Install dependencies: `npm install`
- Lightweight JavaScript syntax check: `npm run check`
- End-to-end tests: `npm run test:e2e`
- Local server: `npm start` serves `http://127.0.0.1:5173`.
- Route/manifest JSON parse sanity check:
  - `node -e "const fs=require('fs'); for (const file of ['routes/catalog.json','routes/candidate-backlog.json','manifest.webmanifest','package.json']) JSON.parse(fs.readFileSync(file,'utf8'))"`
- Current e2e coverage includes:
  - Loading the production JSON catalog and PWA assets.
  - Clean production badges without review metadata.
  - Scenery/terrain badge normalization.
  - Hidden candidate review mode and review decision export.
  - Favorites persistence and favorites-only filter.
  - Search/filter persistence.
  - Continue ride, recents, hero state, and iframe loading.
  - Reset local data.
  - Install prompt handling.
  - Export/import validation.
  - Service-worker offline app shell and route data.

## Social and branding

- OG/Twitter metadata in `index.html` points to `https://pedalscape.com/assets/og-image.png`.
- OG image source asset: `assets/og-image.png`.
- OG image generation script: `scripts/generate-og-image.ps1`.
- Favicon/header icon:
  - `icons/favicon.svg`
  - `icons/app-icon.svg`
  - PNG PWA icons in `icons/icon-192.png`, `icons/icon-512.png`, and `icons/apple-touch-icon.png`

## Known caveats and next queue

- Custom playlists are a later feature; today the app is catalog + selection + continue/favorites/recents.
- There is no formal JSON Schema validation yet; validation is currently syntax checks plus e2e assumptions.
- Continue promoting and curating candidates, but keep the public catalog curated rather than raw/backlog-sized.
- Consider splitting `src/app.js` into modules before the app grows much further or before the catalog/review workflow expands past 100+ reviewed routes.
- YouTube embeds and thumbnails are third-party resources; availability, embed settings, thumbnails, ads, and playback behavior can change outside this repo.
- Keep product naming aligned over time: user-facing brand is PedalScape, while repo/package/backlog text still contains ScenicSpin/scenic-ride-catalog references.
