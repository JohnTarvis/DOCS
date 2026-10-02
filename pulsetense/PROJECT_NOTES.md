> Source of truth: This is the canonical PulseTense project notes file. GitHub URL: https://github.com/JohnTarvis/DOCS/blob/main/pulsetense/PROJECT_NOTES.md

# PulseTense Project Notes

Last updated: 2026-10-02

## Project Layout

- `_new/backend/`: backend API and auth-protected admin routes, with backend-only notes in `_new/backend/documents/BACK-END_NOTES.md`
- `_new/frontend/`: Vite + React frontend
- `_new/frontend/documents/`: active frontend-only planning notes, with older supporting notes grouped under `misc/`
- `_new/backend/documents/sql/`: backend schema and compatibility SQL

## Current State

- Canonical project notes now cover full-site concerns only.
- Frontend-only implementation, UX, validation, and planning notes now live in `_new/frontend/documents/FRONT-END_NOTES.md`.
- Backend-only implementation, schema, and operations notes now live in `_new/backend/documents/BACK-END_NOTES.md`.
- Auth and gallery data are still mid-transition across the full site. The backend cookie path has now been hardened for hosted HTTPS use, but deployed OAuth login is temporarily routed through explicit token-mode redirects because the split Netlify-to-Heroku session flow remains sensitive to browser cross-site cookie policy.
- The gallery stack is also mid-transition to `gallery_v2` through compatibility views and explicit schema endpoints.
- The backend tag path is now being hardened around the active edit and upload flows: existing DB tag display names were lowercased in the live database, new DB-backed tag writes are now intended to store lowercase names end to end, and the compatibility tag-relation layer now tolerates blank placeholder tag entries instead of aborting the whole save. Hosted smoke on 2026-10-02 still found one remaining upload-db defect: brand-new uploads created lowercase tag rows but did not persist `subject_tags` or `set_tags` relations for the new subject or set. Backend investigation has now traced that defect to the upload helper writing relation rows by `tag_id` instead of the compat relation `tag` path; the backend fix is local and still needs deploy plus hosted browser smoke.
- The next major full-site editing milestone is a broader admin edit-system overhaul so subjects, sets, and images can be managed with much finer control.
- The first-pass frontend-only `/edit` overhaul is now in place locally through layout, readability, set-editor usability, uploader clarity, selection or staging preparation, and direct edit-surface tests; the next stage is primarily backend and contract work so the fuller admin controls can become durable.

## Cross-Service Contract

- JWTs must continue to include a server-controlled `role` claim.
- Admin-only routes still accept bearer auth, and now also accept the auth cookie carrying the same claims.
- New session bootstrap endpoints are available for the frontend migration: `GET /api/auth/session`, `GET /api/auth/me`, and `POST /api/auth/logout`.
- Clean OAuth session bootstrap is available now through `GET /api/auth/google?mode=session`, `GET /api/auth/patreon?mode=session`, `GET /api/auth/google/session`, and `GET /api/auth/patreon/session`.
- The default `GET /api/auth/google` and `GET /api/auth/patreon` routes now resolve to the clean session redirect; explicit legacy token redirects remain available only through `?mode=token` for compatibility.
- The deployed frontend login buttons currently use the explicit `?mode=token` OAuth path as a production workaround so Google and Patreon login do not depend on a third-party session cookie surviving the cross-site callback.
- Explicit comparison reads remain available: `GET /api/db?schema=public`, `GET /api/db?schema=gallery_v2`, and `GET /api/db?compare=1`.

## Validation Status

- Detailed frontend validation notes now live in `_new/frontend/documents/FRONT-END_NOTES.md`.
- Detailed backend validation notes now live in `_new/backend/documents/BACK-END_NOTES.md`.
- Backend auth-cookie hardening was validated locally with a focused regression test in `_new/backend/main/__tests__/authSessionUtils.test.js` and deployed to Heroku release `v483`.
- The frontend OAuth entrypoint switch to explicit token mode was validated with targeted ESLint on the touched files and a successful production build before being pushed to `pulse-tense-website-frontend` `main`.
- Live bundle inspection on 2026-09-07 confirmed the hosted frontend now includes both the explicit `?mode=token` OAuth starts and the explicit `?schema=gallery_v2` gallery request path.
- Live browser validation on 2026-09-07 confirmed that Google login succeeds again on the deployed site after the token-mode OAuth switch.
- Live browser validation on 2026-09-07 also confirmed that all 12 `FayeValentine/set1` thumbnails open the matching full-size image with no preview-to-modal filename mismatches.
- Local backend startup resilience was also revalidated on 2026-09-07: missing local OAuth env no longer crashes the API, `/api/health` returns `{"ready":true}` with a temporary `JWT_SECRET`, and missing DB credentials now surface an explicit configuration error instead of opaque SSL or SCRAM startup failures.
- Live backend tag normalization maintenance on 2026-10-02 successfully applied `_new/backend/documents/sql/lowercase-tag-display-names.sql` to Heroku Postgres, lowercased the existing mixed-case tag rows, redefined the compat-layer tag functions, and verified that both `gallery_v2.tags` and `public.tags` now have 0 remaining uppercase tag names.
- Live browser smoke on 2026-10-02 confirmed the hosted token-mode OAuth fallback now works end to end for Google and Patreon admin return paths: the site returned to an authenticated admin state, `/edit` loaded for admin, refresh preserved the session, and logout returned to the public gallery. The VS Code integrated browser still blocked Patreon's embedded Google popup when triggered programmatically, so manual provider completion or a normal browser remains safer for provider-step validation.
- Live upload smoke on 2026-10-02 used the hosted `/edit` uploader to create and then delete a throwaway subject with mixed-case subject and set tags. The upload created the subject, set, and lowercase tag rows, but `GET /api/db?schema=gallery_v2` still returned empty `subject_tags` and `set_tags` relations for that new upload, isolating the remaining new-upload tag bug to the backend upload-to-relation path rather than frontend tag serialization.
- Focused backend follow-up on 2026-10-02 updated the upload relation helper to write normalized tag names through the compat `subject_tags` and `set_tags` `tag` column, and `node --test __tests__/uploadDB_utils.node.test.js` passed 2 out of 2 targeted checks for the new-upload relation path. A deeper local DB smoke still could not run in this checkout because local DB env is not configured here.
- Site-wide validation should continue to verify deployed auth/session behavior, active gallery schema behavior, and admin edit flows together after each milestone.

## Current Follow-Up Items

- If the hosted Patreon flow needs repeated Google-provider validation, use a normal browser rather than the VS Code integrated browser because Patreon’s embedded Google popup can be blocked there even when the token-mode return path itself works.
- If the product goal remains minimal cookie usage, move OAuth mode persistence off the `pt_oauth_mode` cookie and into a signed `state` value or explicit callback variants so the redirect-mode choice no longer depends on any browser cookie at all.
- If clean `httpOnly` session login is still required on split frontend and API origins, treat it as a separate backend architecture task and validate it specifically against modern third-party-cookie restrictions rather than assuming correct `SameSite=None; Secure` attributes are sufficient.
- Populate `_new/backend/main/.env` from `_new/backend/main/.env.example` before expecting local DB-backed routes to work in this checkout; there is no committed local JWT or Postgres config here.
- Retire the remaining browser token-bootstrap compatibility path only after the team decides whether the long-term production auth model is same-site session, cross-site session, or token-first OAuth.
- The site still needs a coordinated edit-system overhaul so admin can manage images, sets, and subjects with finer control across the full stack.
- Deploy the backend `/api/upload-db` relation-write fix for brand-new uploads, then browser-smoke new-set creation where the subject name autofills as the first tag and confirm `GET /api/db?schema=gallery_v2` now returns non-empty `subject_tags` and `set_tags` relations for the new upload.
- `gallery_v2.images` remains an important missing piece for end-to-end image-level editing and should be backfilled from a trusted storage inventory path.
- Backend-specific schema, deployment, and operational follow-up lives in `_new/backend/documents/BACK-END_NOTES.md`.

## Edit-System Next Stage Handoff

- The frontend-only `/edit` patch track is now complete in its first pass, including direct regression coverage for the current edit surface; the next milestone should assume the UI can already focus a subject or set, open a set editor, stage in-modal reorder or removal changes, and surface recent action feedback.
- Keep the full set storage `path` as the canonical set identifier for edit-set, set-tag, and future set-move operations; earlier bugs confirmed that `set1` alone is not stable enough when the backend resolves real storage paths.
- Add a durable backend path for moving one or more existing images between sets without forcing a fresh upload flow; this should update canonical DB image rows, preserve ordering, and stay safe for S3-backed storage.
- Add a durable backend path for adding existing images into an existing set and for creating a new set from selected existing images, not just from brand-new uploads.
- Add a durable backend path for moving a set from one subject to another, including storage-path updates, compatibility-layer updates, and any cascading references that depend on the set path.
- The backend and frontend now both preserve mixed existing tag IDs and newly typed tag names in DB-backed edit saves; browser-smoke one deployed tag edit after release to confirm the hosted frontend matches the local regression coverage.
- The hosted frontend already lowercases `subjectTags` and `setTags` before multipart upload and has direct test coverage for that request shape. The backend upload-db relation-write fix is now local; the next handoff is deploy plus hosted browser smoke before reopening any frontend serialization questions.
- Finish the `gallery_v2.images` backfill and keep image-level IDs authoritative so future edit routes can target canonical image rows instead of inferred filenames alone.
- Replace or redesign `/api/delete-all` for DB or S3-backed mode; the current local-only destructive path is not sufficient for the broader admin editor and should not be treated as production-safe for cloud storage.
- When planning the next backend routes, prefer batch-friendly request shapes so the current frontend staging area can grow into multi-image and multi-set operations without another contract rewrite.

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
- `_new/backend/documents/sql/gallery-v2-schema.sql`
- `_new/backend/documents/sql/gallery-v2-compat.sql`
- `_new/backend/documents/sql/lowercase-tag-display-names.sql`
- `_new/frontend/documents/FRONT-END_NOTES.md`

## Recommendation For Note Format

- Use Markdown as the working source of truth for project notes.
- Generate or update a PDF only when you want a static handoff or archive snapshot.