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

- Canonical project notes now cover full-site concerns only.
- Frontend-only implementation, UX, validation, and planning notes now live in `_new/frontend/documents/FRONT-END_NOTES.md`.
- Backend-only implementation, schema, and operations notes now live in `_new/backend/documents/BACK-END_NOTES.md`.
- Auth and gallery data are mid-transition across the full site: session bootstrap exists on the backend, while the live site still needs the coordinated frontend cutover.
- The gallery stack is also mid-transition to `gallery_v2` through compatibility views and explicit schema endpoints.
- The next major full-site editing milestone is a broader admin edit-system overhaul so subjects, sets, and images can be managed with much finer control.

## Cross-Service Contract

- JWTs must continue to include a server-controlled `role` claim.
- Admin-only routes still accept bearer auth, and now also accept the auth cookie carrying the same claims.
- New session bootstrap endpoints are available for the frontend migration: `GET /api/auth/session`, `GET /api/auth/me`, and `POST /api/auth/logout`.
- Clean OAuth session bootstrap is available now through `GET /api/auth/google?mode=session`, `GET /api/auth/patreon?mode=session`, `GET /api/auth/google/session`, and `GET /api/auth/patreon/session`.
- The existing `GET /api/auth/google` and `GET /api/auth/patreon` routes still default to the legacy token-in-URL redirect until the frontend switches to session bootstrap.
- Explicit comparison reads remain available: `GET /api/db?schema=public`, `GET /api/db?schema=gallery_v2`, and `GET /api/db?compare=1`.

## Validation Status

- Detailed frontend validation notes now live in `_new/frontend/documents/FRONT-END_NOTES.md`.
- Detailed backend validation notes now live in `_new/backend/documents/BACK-END_NOTES.md`.
- Site-wide validation should continue to verify deployed auth/session behavior, active gallery schema behavior, and admin edit flows together after each milestone.

## Current Follow-Up Items

- The legacy `?token=...` OAuth redirect is still the default on the existing `/api/auth/google` and `/api/auth/patreon` routes for compatibility; once the frontend session bootstrap lands, flip the default to clean session redirect and retire the token-in-URL flow.
- The live frontend session cutover still needs to be coordinated with the backend before the legacy token-in-URL redirect can be retired.
- The site still needs a coordinated edit-system overhaul so admin can manage images, sets, and subjects with finer control across the full stack.
- `gallery_v2.images` remains an important missing piece for end-to-end image-level editing and should be backfilled from a trusted storage inventory path.
- Backend-specific schema, deployment, and operational follow-up lives in `_new/backend/documents/BACK-END_NOTES.md`.

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

## Recommendation For Note Format

- Use Markdown as the working source of truth for project notes.
- Generate or update a PDF only when you want a static handoff or archive snapshot.