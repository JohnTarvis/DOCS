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
- The live Postgres database now also contains a separate `gallery_v2` schema for model experiments while the running app remains on the existing `public` schema.
- `gallery_v2` keeps normalized gallery metadata tables and adds a first-class `images` catalog table that does not exist in the current live schema.
- The initial `gallery_v2` seed copied the current relational data and canonicalized one duplicate normalized tag pair: `old-school` and `old school` now map to one v2 tag record.
- The local backend now supports schema-aware gallery metadata reads for frontend testing: `/api/db` keeps the legacy `public` payload, `/api/db?schema=gallery_v2` returns the v2 payload, and `/api/db?compare=1` returns both schemas side by side.

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
- `/api/db` is still assumed to remain public until the gallery data path is redesigned.

## Validation Status

- Live site behavior was manually verified for the frontend auth and edit-flow fixes.
- Live modal-image timing on `https://pulsetense.netlify.app/` was spot-checked in-browser: uncached enlarged images loaded in roughly 1.7 to 2.4 seconds.
- The live modal-image payloads observed in-browser were roughly 2.1 MB to 2.5 MB per image, so the main remaining delay appears to be watermarked image size and delivery rather than frontend modal rendering overhead.
- Focused backend smoke validation for the thumbnail route confirmed repeated requests now reuse cached output across cache-busting `reload` query params and return `304` on matching `If-None-Match`.
- Focused local auth/session smoke validation passed with a stubbed user model: `GET /api/auth/session` anonymous returned `200` with `authenticated: false`; `POST /api/auth/login` returned `200`, preserved the token response, and set an `HttpOnly` auth cookie; `GET /api/auth/session` with the cookie returned `200` with `tokenSource: cookie` and `role: admin`; `GET /api/auth/me` with a bearer token returned `200` with `tokenSource: bearer`; cookie-protected auth and admin routes returned `200`; and `POST /api/auth/logout` returned `200` and cleared the auth cookie.
- Local redirect-mode helper validation passed: session mode builds a clean frontend redirect without `?token=...`, while legacy mode still builds the token-in-URL redirect for compatibility.
- `gallery_v2` creation was validated in the live database: 30 `subjects`, 32 `sets`, 61 canonical `tags`, 99 `subject_tags`, 10 `set_tags`, 2 `users`, and an empty `images` table ready for backfill.
- The updated `/api/db` route was smoke-tested locally against a stubbed DB layer and then against the live database with `DATABASE_URL` set for the local process.
- The live-db smoke test returned `200` for `/db?compare=1` and `200` for `/db?schema=gallery_v2`, with counts matching the expected split: `public` has 62 tags, while `gallery_v2` has 61 canonical tags and 0 images.
- Focused regression coverage was added for auth state, edit helpers, tag filtering, and thumbnail-first modal loading.
- The gallery Jest harness was updated so the focused gallery tests run cleanly.
- Full Jest passed during the recent tagging work.
- Production build passed during the recent auth, edit-surface, tagging, modal-loading, and banner work.
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
- The backend still queries unqualified `public` table names, so `gallery_v2` remains an experimental schema until code adds schema qualification or a dedicated search path.
- Only `/api/db` is schema-aware right now; the rest of the DB-backed read and edit routes still assume `public` and would need the same abstraction before end-to-end v2 testing through the full backend surface.
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
- Backfill `gallery_v2.images` from a trusted S3 inventory or direct S3 listing path so image identity stops being inferred at request time.
- If the frontend starts consuming v2 payloads, factor schema selection into a shared data-access layer so `images-db`, edit routes, and any future set or tag routes do not duplicate schema-specific SQL by hand.
- Decide whether v2 adoption should happen through schema-qualified queries, a configurable search path, or a staged data-migration layer in the backend.
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

Schema source for recreation or review lives in `documents/gallery-v2-schema.sql`.

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