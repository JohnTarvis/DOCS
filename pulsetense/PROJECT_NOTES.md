> Source of truth: This is the canonical PulseTense project notes file. GitHub URL: https://github.com/JohnTarvis/DOCS/blob/main/pulsetense/PROJECT_NOTES.md

# PulseTense Project Notes

Last updated: 2026-09-06

## Project Layout

- `_new/backend/`: backend API and auth-protected admin routes
- `_new/frontend/`: Vite + React frontend
- `_new/frontend/documents/`: detailed frontend architecture, bug triage, and handoff notes

## Current State

- The live frontend auth and `/edit` admin flow are fixed and verified working on the website.
- Frontend admin gating now uses the JWT `role` claim via structured auth state and a derived `isAdmin` flag.
- The frontend now strips OAuth callback tokens before analytics loads, keeps auth persistence in `sessionStorage` instead of `localStorage`, and applies a stricter cross-origin referrer policy from the document shell.
- The edit surface uses authenticated requests against `VITE_API_URL` for upload, edit-set, delete-set, and delete-all operations.
- The edit surface now refreshes the shared gallery state in place after mutations instead of forcing full page reloads.
- Subject-tag editing on the edit surface now resolves the correct `subject_id` and preserves comma-separated tag input.
- Backend thumbnail responses now reuse generated thumbnails in-process, reuse the loaded watermark buffer, and send `Cache-Control` plus `ETag` headers.
- The backend now has additive cookie-session primitives alongside the existing bearer flow: `POST /api/auth/login` still returns a token, also sets an auth cookie, auth middleware accepts either bearer or cookie auth, and `GET /api/auth/session`, `GET /api/auth/me`, and `POST /api/auth/logout` are now implemented.
- Google and Patreon OAuth now support clean cookie-session bootstrap through `?mode=session` and dedicated `/api/auth/google/session` and `/api/auth/patreon/session` entrypoints.
- The existing `/api/auth/google` and `/api/auth/patreon` entrypoints still default to the legacy `?token=...` redirect so the currently working frontend does not regress before it switches to session bootstrap.
- Local username/password registration now writes `password_hash` to match the live `users` schema instead of the broken old `password` column expectation.
- Gallery and carousel modal views now show the existing thumbnail stretched to the modal frame immediately while the larger image loads, replacing the old spinner-only wait state.
- The top banner now uses a darker cinematic gradient and an animated logo treatment with soft red light rays behind the emblem.
- The admin Test Page is now repurposed as a schema comparison surface that fetches both `/api/db` and `/api/db?schema=gallery_v2`, samples random gallery images, and times preview plus full-image loading side by side.
- The live Postgres database now contains both normalized `gallery_v2` base tables and a `gallery_v2_compat` view-and-trigger layer that maps the existing legacy backend SQL shape into v2.
- `gallery_v2` keeps normalized gallery metadata tables and adds a first-class `images` catalog table that does not exist in the current live schema.
- The initial `gallery_v2` seed copied the current relational data and canonicalized one duplicate normalized tag pair: `old-school` and `old school` now map to one v2 tag record.
- The backend DB pool now defaults its search path to `gallery_v2_compat,gallery_v2,public`, so unqualified gallery metadata reads and the current legacy upload/edit SQL resolve to v2-backed tables without changing the frontend contract.
- `/api/db` now returns the active legacy-shaped v2-backed payload by default, `/api/db?schema=public` returns the original `public` payload, `/api/db?schema=gallery_v2` returns the raw v2 payload, and `/api/db?compare=1` returns both explicit schemas side by side for the comparison page.

## Tag System Updates

- Tag interactions now use left click to include and right click to exclude.
- Included tags turn green; excluded tags turn red.
- Non-active tags remain visible instead of disappearing.
- Coinciding tags show a subtle light green hint.
- Non-coinciding tags show a subtle light red hint.
- Active tags no longer jump to the top of the list when clicked.
- Active tags no longer change font size when clicked.

## Backend Contract The Frontend Now Relies On

- JWTs must continue to include a server-controlled `role` claim.
- Admin-only routes still accept bearer auth, and now also accept the auth cookie carrying the same claims.
- New session bootstrap endpoints are available for the frontend migration: `GET /api/auth/session`, `GET /api/auth/me`, and `POST /api/auth/logout`.
- Clean OAuth session bootstrap is available now through `GET /api/auth/google?mode=session`, `GET /api/auth/patreon?mode=session`, `GET /api/auth/google/session`, and `GET /api/auth/patreon/session`.
- The existing `GET /api/auth/google` and `GET /api/auth/patreon` routes still default to the legacy token-in-URL redirect until the frontend switches to session bootstrap.
- The main frontend gallery path still reads a legacy-shaped payload from `/api/db`, but that default payload is now backed by `gallery_v2` through the compatibility layer rather than `public`.
- Explicit comparison reads remain available: `GET /api/db?schema=public`, `GET /api/db?schema=gallery_v2`, and `GET /api/db?compare=1`.

## Validation Status

- Live site behavior was manually verified for the frontend auth and edit-flow fixes.
- Live modal-image timing on `https://pulsetense.netlify.app/` was spot-checked in-browser: uncached enlarged images loaded in roughly 1.7 to 2.4 seconds.
- The live modal-image payloads observed in-browser were roughly 2.1 MB to 2.5 MB per image, so the main remaining delay appears to be watermarked image size and delivery rather than frontend modal rendering overhead.
- Focused backend smoke validation for the thumbnail route confirmed repeated requests now reuse cached output across cache-busting `reload` query params and return `304` on matching `If-None-Match`.
- Focused local auth/session smoke validation passed with a stubbed user model: `GET /api/auth/session` anonymous returned `200` with `authenticated: false`; `POST /api/auth/login` returned `200`, preserved the token response, and set an `HttpOnly` auth cookie; `GET /api/auth/session` with the cookie returned `200` with `tokenSource: cookie` and `role: admin`; `GET /api/auth/me` with a bearer token returned `200` with `tokenSource: bearer`; cookie-protected auth and admin routes returned `200`; and `POST /api/auth/logout` returned `200` and cleared the auth cookie.
- Local redirect-mode helper validation passed: session mode builds a clean frontend redirect without `?token=...`, while legacy mode still builds the token-in-URL redirect for compatibility.
- `gallery_v2` creation was validated in the live database: 30 `subjects`, 32 `sets`, 61 canonical `tags`, 99 `subject_tags`, 10 `set_tags`, 2 `users`, and an empty `images` table ready for backfill.
- The new `gallery_v2_compat` schema was applied successfully in the live database from `documents/gallery-v2-compat.sql`.
- The updated `/api/db` route was smoke-tested locally against a stubbed DB layer and then against the live database with `DATABASE_URL` set for the local process.
- The live-db cutover smoke showed the backend pool using `search_path = gallery_v2_compat,gallery_v2,public`.
- The live-db cutover smoke returned the new default `/api/db` payload counts from the active v2-backed layer: 30 `subjects`, 32 `sets`, 61 `tags`, 10 `set_tags`, and 99 `subject_tags`.
- The same live-db smoke confirmed explicit comparison reads still match expectations: `GET /api/db?schema=public` returned 62 tags, while `GET /api/db?schema=gallery_v2` returned 61 canonical tags and 0 images.
- Representative legacy write SQL was exercised successfully against the compatibility layer inside rollback transactions: `UPDATE sets SET name = name ...`, `INSERT INTO set_tags (set_id, tag) ...`, `INSERT INTO subject_tags (subject_id, tag) ...`, and upload-style `INSERT INTO subjects (...)` plus `INSERT INTO sets (...)` all resolved cleanly into `gallery_v2` and left no persisted smoke-test data.
- The new admin Test Page comparison was browser-validated locally against the live API through a Vite proxy target. In one sampled run, both schemas reported 32 sets and 891 generated images, and both metadata requests completed in 363 ms.
- That same sampled run showed image timing dominated by asset variance rather than schema selection: original-schema previews that completed landed around 1052 to 1061 ms with full images around 4260 to 4580 ms, while `gallery_v2` preview timings ranged from 1057 to 4014 ms and full-image timings ranged from 1770 to 7749 ms.
- Focused regression coverage was added for auth state, edit helpers, tag filtering, and thumbnail-first modal loading.
- The gallery Jest harness was updated so the focused gallery tests run cleanly.
- Full Jest passed during the recent tagging work.
- Production build passed during the recent auth, edit-surface, tagging, modal-loading, banner, and schema-test-page work.
- The updated top banner was browser-verified locally against mocked `/api/health` and `/api/db` responses because the local backend health gate was unavailable.
- Repo-wide lint still has unrelated pre-existing failures outside the recent auth/edit/tagging changes.
- The backend repository still lacks a runnable local `jest` binary, so the committed thumbnail endpoint regression test cannot yet run through `npm test`.

## Current Follow-Up Items

- The gallery modal tag editor path should be realigned with the current `EditTagsModal` API.
- Backend cookie-session primitives are now in place, but the live frontend has not yet switched its auth bootstrap from token/sessionStorage handling to `GET /api/auth/session` or `GET /api/auth/me`.
- The legacy `?token=...` OAuth redirect is still the default on the existing `/api/auth/google` and `/api/auth/patreon` routes for compatibility; once the frontend session bootstrap lands, flip the default to clean session redirect and retire the token-in-URL flow.
- Add a cookie policy page or footer link and keep it aligned with Google Analytics usage plus any future auth or session cookie behavior.
- The modal currently downloads full watermarked images for first paint; a dedicated modal-sized variant or more compressed format would likely be the biggest remaining image-load win.
- Thumbnail caching is now improved within a single backend process, but persistent cache reuse across dyno restarts or multiple instances is still open.
- `gallery_v2.images` is intentionally unseeded for now because the current live schema does not catalog images yet; a later backfill should derive canonical image rows from a trusted S3 inventory path rather than the current ad hoc API shape.
- Image-level DB routes are still limited by the empty `gallery_v2.images` catalog, so paths such as `/api/delete-image-db` are not meaningfully migrated until image backfill lands or those routes are retired.
- The live database layer is migrated now, but the hosted backend will not use the new v2-default search path until this code is deployed.
- The schema comparison Test Page currently samples random images, so repeated runs or a matched-image mode would make public-vs-v2 timing comparisons less noisy.
- Red-tag exclusion currently depends on right click; a mobile-safe exclusion affordance is still worth adding.
- There are stale tag utilities and duplicate context files that should either be removed or realigned.
- Local backend env and session settings still need one explicit reference note for browser validation: `JWT_SECRET`, `DATABASE_URL`, `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`, `PATREON_CLIENT_ID`, `PATREON_CLIENT_SECRET`, `PATREON_REDIRECT_URI`, `FRONTEND_URL`, `AUTH_COOKIE_NAME`, `AUTH_TOKEN_EXPIRES_IN`, `AUTH_COOKIE_SECURE`, and `OAUTH_REDIRECT_MODE`.

## Backend Agent Recommendations

- Stop redirecting the frontend with a bearer token in the query string; prefer a short-lived one-time code exchange or a server-set `httpOnly`, `Secure`, `SameSite` cookie.
- If auth moves to cookies, add CSRF protection for state-changing admin routes instead of relying on bearer-token possession alone.
- Send security headers from the deployed API and edge layer, especially `Content-Security-Policy`, `Referrer-Policy`, `X-Content-Type-Options`, and `Strict-Transport-Security`.
- Keep CORS restricted to the real frontend origin set, and continue enforcing admin authorization fully on the backend even if the frontend hides edit controls.
- Keep the backend OAuth/client secrets and database TLS settings documented for local development so browser validation is not blocked by missing env or SSL mismatches.

## Backend Next Steps

- Priority 1 is now partially staged: the backend cookie-session primitives and session endpoints exist, but the frontend cutover and the default clean OAuth redirect are still pending.
- Recommended cutover sequence: switch the frontend auth bootstrap to `GET /api/auth/session` or `GET /api/auth/me`, move OAuth starts to `?mode=session` or the dedicated session entrypoints, then change the backend default redirect mode to clean session redirect and retire the token-in-URL flow.
- `POST /api/auth/logout` now exists to clear the auth cookie server-side.
- If cookie auth becomes the primary live path for admin mutations such as upload, edit-set, delete-set, and delete-all, add CSRF protection for those credentialed state-changing routes.
- Keep the backend role claim or equivalent server-side admin check as the source of truth; frontend role gating must remain cosmetic only.
- Apply production security headers at the backend or platform edge: `Content-Security-Policy`, `Referrer-Policy`, `X-Content-Type-Options`, `Strict-Transport-Security`, and `Permissions-Policy` where appropriate.
- Reconfirm CORS after the auth change so only the real frontend origins are allowed and credentialed requests are intentional.
- Extend thumbnail caching beyond in-process memory if live measurements still show costly first-hit regeneration after deploys or instance churn.
- Deploy the backend build that includes the `gallery_v2_compat` search-path cutover, then browser-smoke the live admin upload, edit-set, delete-set, and tag-edit flows against production.
- Backfill `gallery_v2.images` from a trusted S3 inventory or direct S3 listing path so image identity stops being inferred at request time, then either migrate image-level routes onto that catalog or remove the DB image table assumption entirely.
- Add focused regression coverage for the compat-backed upload/edit/delete DB routes once Jest is installed locally in the backend repo.
- Review the modal image path from the backend side as a performance and exposure issue: serve a dedicated modal-sized watermarked asset rather than the largest watermarked file for first paint when possible.
- Install and pin Jest in the backend repo so committed endpoint regression tests can run through the standard test command.
- Document the required local backend env for JWT, OAuth, and database TLS so local end-to-end browser validation can start without manual discovery.

## Gallery V2 Tables

- `gallery_v2.subjects`
- `gallery_v2.sets`
- `gallery_v2.images`
- `gallery_v2.tags`
- `gallery_v2.subject_tags`
- `gallery_v2.set_tags`
- `gallery_v2.users`

## Gallery V2 Compatibility Views

- `gallery_v2_compat.subjects`
- `gallery_v2_compat.sets`
- `gallery_v2_compat.tags`
- `gallery_v2_compat.subject_tags`
- `gallery_v2_compat.set_tags`

Schema source for recreation or review lives in `documents/gallery-v2-schema.sql` and `documents/gallery-v2-compat.sql`.

## Frontend Agent Recommendations

- Keep the early auth bootstrap ahead of analytics so OAuth callback tokens are scrubbed before third-party scripts load.
- Do not reintroduce `localStorage` token persistence; keep browser-managed auth in `sessionStorage` until the backend owns the session fully.
- Keep frontend admin gating cosmetic only and continue assuming all real authorization is enforced by the backend.
- Keep API calls on the shared API-base pattern and avoid drift back to ad hoc relative `/api` fetches.
- Add a visible cookie policy or privacy disclosure link in the frontend once the policy text is ready.

## Frontend Next Steps

- Add a cookie policy page and footer or login-surface link that documents Google Analytics plus any future auth or session cookies.
- When the backend exposes a session endpoint, remove browser token handling from the frontend auth bootstrap and read auth state from `GET /api/auth/session` or `GET /api/auth/me` instead.
- Re-test live OAuth after the backend session migration to confirm the URL no longer exposes tokens and logout clears the server-owned session correctly.
- Realign the gallery modal tag editor path with the current `EditTagsModal` interface.
- Keep the modal image-load follow-up in scope once the backend can serve a modal-sized watermarked asset for first paint.

## Detailed Source Notes

- `frontend/documents/frontend-agent-handoff-2026-09-05.md`
- `frontend/documents/frontend-bugs-and-fixes.md`
- `frontend/documents/planned-frontend-updates.md`
- `frontend/documents/frontend-architecture.md`

## Recommendation For Note Format

- Use Markdown as the working source of truth for project notes.
- Generate or update a PDF only when you want a static handoff or archive snapshot.