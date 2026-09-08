> Source of truth: This is the canonical PulseTense project notes file. GitHub URL: https://github.com/JohnTarvis/DOCS/blob/main/pulsetense/PROJECT_NOTES.md

# PulseTense Project Notes

Last updated: 2026-09-07

## Project Layout

- `_new/backend/`: backend API and auth-protected admin routes, with backend-only notes in `_new/backend/documents/BACK-END_NOTES.md`
- `_new/frontend/`: Vite + React frontend
- `_new/frontend/documents/`: active frontend-only planning notes, with older supporting notes grouped under `misc/`
- `_new/backend/documents/sql/`: backend schema and compatibility SQL
- `_new/backend/documents/misc/`: older backend reviews and supporting notes

## Current State

- The live frontend auth and `/edit` admin flow are fixed and verified working on the website.
- Frontend admin gating now uses the JWT `role` claim via structured auth state and a derived `isAdmin` flag.
- The frontend now strips OAuth callback tokens before analytics loads, keeps auth persistence in `sessionStorage` instead of `localStorage`, and applies a stricter cross-origin referrer policy from the document shell.
- The edit surface uses authenticated requests against `VITE_API_URL` for upload, edit-set, delete-set, and delete-all operations.
- The edit surface now refreshes the shared gallery state in place after mutations instead of forcing full page reloads.
- Subject-tag editing on the edit surface now resolves the correct `subject_id` and preserves comma-separated tag input.
- The edit surface now also sends full set storage paths for set-tag and set-order saves, keeps the tag modal on the shared API-base pattern, and preserves all-image deletions from the set editor instead of silently dropping an empty reorder request.
- Gallery and carousel modal views now show the existing thumbnail stretched to the modal frame immediately while the larger image loads, replacing the old spinner-only wait state.
- The top banner now uses a darker cinematic gradient and an animated logo treatment with soft red light rays behind the emblem.
- The main frontend gallery loader now explicitly requests `GET /api/db?schema=${VITE_GALLERY_DB_SCHEMA}` and normalizes the configured schema payload into the existing client-side gallery contract; the current migrated path targets `gallery_v2`.
- The admin Test Page is now repurposed as a schema comparison surface that fetches `GET /api/db?schema=public` and `GET /api/db?schema=gallery_v2`, samples random gallery images, and times preview plus full-image loading side by side.
- The frontend gallery transform now prefers canonical `gallery_v2.images` rows when they exist and falls back to inferred filenames only while the image catalog remains empty.
- Backend-only implementation, schema, and operations notes now live in `_new/backend/documents/BACK-END_NOTES.md`.
- The next planned frontend task is a broader `/edit` page overhaul focused on admin control, better readability, and clearer editing workflows for subjects, sets, and images.

## Tag System Updates

- Tag interactions now use left click to include and right click to exclude.
- Included tags turn green; excluded tags turn red.
- Non-active tags remain visible instead of disappearing.
- Coinciding tags show a subtle light green hint.
- Non-coinciding tags show a subtle light red hint.
- Active tags no longer jump to the top of the list when clicked.
- Active tags no longer change font size when clicked.

## Cross-Service Contract

- JWTs must continue to include a server-controlled `role` claim.
- Admin-only routes still accept bearer auth, and now also accept the auth cookie carrying the same claims.
- New session bootstrap endpoints are available for the frontend migration: `GET /api/auth/session`, `GET /api/auth/me`, and `POST /api/auth/logout`.
- Clean OAuth session bootstrap is available now through `GET /api/auth/google?mode=session`, `GET /api/auth/patreon?mode=session`, `GET /api/auth/google/session`, and `GET /api/auth/patreon/session`.
- The existing `GET /api/auth/google` and `GET /api/auth/patreon` routes still default to the legacy token-in-URL redirect until the frontend switches to session bootstrap.
- The main frontend gallery path now explicitly reads `GET /api/db?schema=gallery_v2` and normalizes the raw v2 payload client-side.
- Explicit comparison reads remain available: `GET /api/db?schema=public`, `GET /api/db?schema=gallery_v2`, and `GET /api/db?compare=1`.

## Validation Status

- Live site behavior was manually verified for the frontend auth and edit-flow fixes.
- Live modal-image timing on `https://pulsetense.netlify.app/` was spot-checked in-browser: uncached enlarged images loaded in roughly 1.7 to 2.4 seconds.
- The live modal-image payloads observed in-browser were roughly 2.1 MB to 2.5 MB per image, so the main remaining delay appears to be watermarked image size and delivery rather than frontend modal rendering overhead.
- Local browser verification against a Vite proxy to the live API confirmed the real gallery page now requests `GET /api/db?schema=gallery_v2`, received `200` with 30 `subjects`, 32 `sets`, and 61 `tags`, and rendered 18 images on pages view without a health-gate error.
- The new admin Test Page comparison was browser-validated locally against the live API through a Vite proxy target. In one sampled run, both schemas reported 32 sets and 891 generated images, and both metadata requests completed in 363 ms.
- That same sampled run showed image timing dominated by asset variance rather than schema selection: original-schema previews that completed landed around 1052 to 1061 ms with full images around 4260 to 4580 ms, while `gallery_v2` preview timings ranged from 1057 to 4014 ms and full-image timings ranged from 1770 to 7749 ms.
- Final frontend close-out validation passed after the explicit schema-v2 migration: full Jest passed with 11 suites and 36 tests, targeted ESLint passed on the changed frontend files, and the production build passed.
- Focused review follow-up validation on 2026-09-07 passed for `src/__tests__/galleryUtils.test.js`, including new coverage for canonical `gallery_v2.images` rows.
- Targeted ESLint passed on the changed frontend edit and gallery-transform files after the 2026-09-07 review patch.
- Static diagnostics found no errors in the touched frontend files after the 2026-09-07 review patch.
- The updated top banner was browser-verified locally against mocked `/api/health` and `/api/db` responses because the local backend health gate was unavailable.

## Current Follow-Up Items

- The gallery modal tag editor path should be realigned with the current `EditTagsModal` API.
- The live frontend has not yet switched its auth bootstrap from token/sessionStorage handling to `GET /api/auth/session` or `GET /api/auth/me`.
- The legacy `?token=...` OAuth redirect is still the default on the existing `/api/auth/google` and `/api/auth/patreon` routes for compatibility; once the frontend session bootstrap lands, flip the default to clean session redirect and retire the token-in-URL flow.
- Add a cookie policy page or footer link and keep it aligned with Google Analytics usage plus any future auth or session cookie behavior.
- The main frontend gallery path is migrated locally now, but the hosted website will not use the explicit `?schema=gallery_v2` request until this frontend build is deployed.
- The schema comparison Test Page currently samples random images, so repeated runs or a matched-image mode would make public-vs-v2 timing comparisons less noisy.
- Red-tag exclusion currently depends on right click; a mobile-safe exclusion affordance is still worth adding.
- The edit-tag modal still drops newly typed tag names because the frontend save path collapses modal input back to numeric tag IDs only; either support string tag names end-to-end through the compat layer or block unknown tags explicitly in the UI.
- `src/components/edit/ImageActions.jsx` remains unused dead code and should either be removed or wired into a real image-level edit path.
- Frontend-only planning for the edit-page overhaul now lives in `frontend/documents/FRONT-END_NOTES.md`, while older frontend handoff and architecture notes are grouped under `frontend/documents/misc/`.
- Backend-specific schema, deployment, and operational follow-up now lives in `_new/backend/documents/BACK-END_NOTES.md`.

## Frontend Agent Recommendations

- Keep the early auth bootstrap ahead of analytics so OAuth callback tokens are scrubbed before third-party scripts load.
- Do not reintroduce `localStorage` token persistence; keep browser-managed auth in `sessionStorage` until the backend owns the session fully.
- Keep frontend admin gating cosmetic only and continue assuming all real authorization is enforced by the backend.
- Keep API calls on the shared API-base pattern and avoid drift back to ad hoc relative `/api` fetches.
- Add a visible cookie policy or privacy disclosure link in the frontend once the policy text is ready.

## Frontend Next Steps

- Make the edit page the next active frontend task and redesign it for much stronger admin control over subject, set, and image editing.
- Frontend goal for the edit overhaul: the admin should be able to add or remove individual images within sets, move images between sets, create sets from selected images, and add, remove, or reassign sets across subjects.
- Improve edit-page readability by replacing the current dense table-first layout with clearer sections, labeled actions, stronger selection states, and a visible staged-changes area.
- Improve edit-page look and usability with better spacing, stronger visual hierarchy, explicit action labels, clearer destructive-action separation, and page-level status feedback.
- Add a cookie policy page and footer or login-surface link that documents Google Analytics plus any future auth or session cookies.
- Switch the frontend auth bootstrap to `GET /api/auth/session` or `GET /api/auth/me` and remove browser token handling from the auth bootstrap.
- Re-test live OAuth after the frontend session cutover and backend default redirect flip to confirm the URL no longer exposes tokens and logout clears the server-owned session correctly.
- Realign the gallery modal tag editor path with the current `EditTagsModal` interface.
- Keep the modal image-load follow-up in scope once the backend can serve a modal-sized watermarked asset for first paint.

## Monetization Recommendation

- Treat original or clearly licensable collections as the primary revenue surface.
- Treat fanart as a discovery surface, not the core commercial catalog.
- Put stronger conversion paths on original-work pages: Patreon tiers, curated digital packs, wallpaper bundles, commissions, and commercial licensing inquiry links.
- Keep fanart monetization conservative: avoid direct print, download, licensing, or high-pressure purchase CTAs on recognizable third-party IP pages unless rights are secured.
- If ads are introduced, place them on the broader public site with the assumption that ad-supported fanart is still commercial use and does not remove copyright or trademark risk.
- Prioritize one low-friction funnel first: a support membership or email capture tied to original-work rewards performs better than scattering weak CTAs across thin pages.

## Fanart Advertising Guidance

- Advertising around fanart can still create the same core licensing problem because the site is commercially benefiting from copyrighted or trademarked characters and brands.
- Ad revenue is usually less aggressive than selling prints or direct licenses, but it is not a clean workaround and can still draw takedowns, trademark complaints, or ad-network policy problems.
- The safer operating model is to separate original/licensable work from fanart in both site structure and revenue strategy.
- Avoid language that implies endorsement, partnership, or official affiliation on fanart pages.
- Keep a visible takedown path and be prepared to remove or de-monetize specific fanart pages quickly if needed.
- If the business model needs dependable monetization, build it around original collections and treat any fanart traffic as non-core and higher risk.

## Detailed Source Notes

- `_new/backend/documents/BACK-END_NOTES.md`
- `_new/backend/documents/misc/backend-code-review-2026-09-05.md`
- `_new/backend/documents/sql/gallery-v2-schema.sql`
- `_new/backend/documents/sql/gallery-v2-compat.sql`
- `frontend/documents/FRONT-END_NOTES.md`
- `frontend/documents/misc/frontend-agent-handoff-2026-09-05.md`
- `frontend/documents/misc/frontend-bugs-and-fixes.md`
- `frontend/documents/misc/planned-frontend-updates.md`
- `frontend/documents/misc/frontend-architecture.md`

## Recommendation For Note Format

- Use Markdown as the working source of truth for project notes.
- Generate or update a PDF only when you want a static handoff or archive snapshot.