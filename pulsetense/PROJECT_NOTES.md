> Source of truth: This is the canonical PulseTense project notes file. GitHub URL: https://github.com/JohnTarvis/DOCS/blob/main/pulsetense/PROJECT_NOTES.md

# PulseTense Project Notes

Last updated: 2026-11-18

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
- The gallery stack is also mid-transition to `gallery_v2` through compatibility views and explicit schema endpoints, but the live `gallery_v2.images` catalog is now populated from the trusted S3-backed storage inventory instead of remaining empty.
- On 2026-11-18, the backend hardened the `edit-set-db` and DB-backed delete flows: edit payloads are validated against the current S3 inventory before mutations, the full catalog replacement uses a dedicated Postgres transaction client, and set/image deletes coordinate storage removal with metadata updates.
- Cookie-authenticated admin mutations now require a double-submit CSRF token: read the readable `pt_csrf` cookie and send the same value as `X-CSRF-Token`. Bearer-authenticated mutations remain supported and do not require this header.
- `GET /api/images-db` has now been retired from the backend surface, and `POST /api/delete-image-db` now uses `gallery_v2.images` as its source of truth for image-level deletes.
- The backend tag path is now being hardened around the active edit and upload flows: existing DB tag display names were lowercased in the live database, new DB-backed tag writes are now intended to store lowercase names end to end, and the compatibility tag-relation layer now tolerates blank placeholder tag entries instead of aborting the whole save. Hosted smoke on 2026-10-02 found one remaining upload-db defect where brand-new uploads created lowercase tag rows but did not persist `subject_tags` or `set_tags` relations for the new subject or set. Backend investigation traced that defect to the upload helper writing relation rows by `tag_id` instead of the compat relation `tag` path, and the deployed backend now passes a hosted browser smoke for that fix: a brand-new upload created the subject, set, lowercase tags, and non-empty relation rows, then cleaned up successfully.
- The next major full-site editing milestone is a broader admin edit-system overhaul so subjects, sets, and images can be managed with much finer control.
- The first-pass frontend-only `/edit` overhaul is now in place locally through layout, readability, set-editor usability, uploader clarity, selection or staging preparation, and direct edit-surface tests; the next stage remains primarily backend and contract work so the fuller admin controls can become durable, with the image catalog and image-delete migration now in place as prerequisites rather than blockers.

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
- Hosted post-deploy upload smoke on 2026-10-02 then confirmed the fix end to end: a fresh throwaway upload through the live `/edit` page showed the expected subject and set tags in the management card, and a direct hosted `GET /api/db?schema=gallery_v2` check returned non-empty `subject_tags` and `set_tags` relation rows for that upload before the throwaway subject was deleted cleanly.
- Focused backend Node smoke on 2026-10-16 validated the new `gallery_v2` image backfill mapper, including nested `wm/` derivatives, root-level `wm_` derivatives, and the missing-bucket CLI failure path.
- A live dry run of `npm run backfill:gallery-images` on 2026-10-16, using Heroku-provided DB and S3 config, reported 35 current sets, 625 catalog image rows, and 0 unmatched S3 keys under `main/uploads/`.
- The live `npm run backfill:gallery-images -- --apply` run on 2026-10-16 then populated `gallery_v2.images` with 625 rows, and a direct verification query confirmed ordered catalog rows with `id`, `set_id`, `filename`, `original_key`, and `sort_order`.
- Focused backend Node smoke on 2026-10-16 also validated the migrated `POST /api/delete-image-db` path: it deletes cataloged storage keys, removes the image row from `gallery_v2.images`, refreshes the parent set `image_count`, and invalidates cached thumbnails by set prefix.
- Mounted route inspection on 2026-10-16 confirmed `GET /api/images-db` is no longer exposed by the backend.
- Backend releases `v502` and `v503` on 2026-10-21 made the live `/edit` flows keep `gallery_v2.images` authoritative with no frontend contract change. Before them, editor deletes and reorders through `POST /api/edit-set-db`, plus new uploads through `POST /api/upload-db`, changed S3 and `image_count` without updating catalog rows. Because the frontend renders catalog rows first, an editor delete could have shown a broken image. Both routes now rebuild the affected set's catalog rows from S3, and `delete-set-db` now clears that set's cached thumbnails.
- A hosted browser smoke on 2026-10-21 passed upload, editor removal, reorder, and set deletion end to end on a disposable `zzsmoketest` subject. Details are in `_new/backend/documents/BACK-END_NOTES.md`. Production finished with 625 catalog images across 35 sets and 0 count/catalog mismatches.
- Site-wide validation should continue to verify deployed auth/session behavior, active gallery schema behavior, and admin edit flows together after each milestone.
- Final backend validation on 2026-11-18 passed Jest 9 suites / 14 tests and the Node-native suite 17/17.

## Current Follow-Up Items

- Deferred until the end, and optional pending a product decision: catalog-backed admin edit routes for moving images between sets, seeding sets from existing images, and moving sets between subjects. Catalog row IDs are regenerated whenever a set is edited, so any future ID-based feature must account for that.
- Decide whether `POST /api/delete-image-db` should be wired into the frontend for single-image deletes or retired. It works, but no UI calls it; the set editor's delete path through `edit-set-db` is now catalog-safe.
- Frontend handoff: cookie-authenticated login, session bootstrap, upload, edit-set, delete-set, delete-image, and logout flows must read `pt_csrf` and send `X-CSRF-Token` on mutation/logout requests. The frontend must not use the cookie value to mutate server state without the header.
- Frontend handoff: decide whether `/edit` should expose `POST /api/delete-image-db` for catalog-ID single-image deletion or continue using `POST /api/edit-set-db` only. The backend route is working but the confirmation dialog, refresh behavior, and user-facing fallback need a frontend product decision.
- Frontend handoff: do not expose `POST /api/delete-all` yet. It remains disabled pending a storage scope, confirmation UX, audit logging, and compensation/rollback design.
- Frontend handoff: derivative keys are currently null in `gallery_v2.images`; a modal/thumbnail asset contract must be chosen before consuming `thumbnail_key`, `modal_key`, or `watermarked_key`.
- Optional frontend handoff: in `EditSetModal`, keep the saving state until the parent `onClose` refresh finishes. Today Save is re-enabled with stale filenames during the gallery refetch. The backend rejects a second click with a 400 order mismatch, so this is UX only.
- Optional frontend handoff: selecting an upload folder overwrites the Subject field with the folder name. Typing a subject also appends it to Subject Tags, which can duplicate a tag already entered.
- Replace or redesign `POST /api/delete-all` as a storage-aware and DB-aware admin operation before any production destructive bulk-delete flow is reintroduced.
- Decide the long-term production auth model, then retire the remaining browser token-bootstrap compatibility path at the right time.
- If the product goal remains minimal cookie usage, move OAuth mode persistence off the `pt_oauth_mode` cookie and into a signed `state` value or explicit callback variants; if cookie auth becomes primary for admin writes, add CSRF protection and recheck credentialed CORS.
- Add a derivative-asset strategy for `gallery_v2.images`: decide when to populate `thumbnail_key`, `modal_key`, and `watermarked_key`, and use that work to reduce modal first-paint cost.
- Decide whether `public` becomes a frozen reference snapshot or receives an explicit one-way sync from `gallery_v2`, and add a deliberate drift check so the two schemas do not silently diverge.
- Keep one light browser smoke on future backend upload, delete-image-db, edit-set, and tag-edit changes; use a normal browser rather than the VS Code integrated browser when Patreon’s embedded Google popup needs repeated validation.
- Populate `_new/backend/main/.env` from `_new/backend/main/.env.example` before expecting local DB-backed routes or the image-backfill CLI to work in a fresh checkout; there is no committed local JWT, Postgres, or S3 config here.
- Backend-specific schema, deployment, and operational follow-up lives in `_new/backend/documents/BACK-END_NOTES.md`.

## Edit-System Next Stage Handoff

- The frontend-only `/edit` patch track is now complete in its first pass, including direct regression coverage for the current edit surface; the next milestone should assume the UI can already focus a subject or set, open a set editor, stage in-modal reorder or removal changes, and surface recent action feedback.
- Keep the full set storage `path` as the canonical set identifier for edit-set, set-tag, and future set-move operations; earlier bugs confirmed that `set1` alone is not stable enough when the backend resolves real storage paths.
- Add a durable backend path for moving one or more existing images between sets without forcing a fresh upload flow; this should update canonical DB image rows, preserve ordering, and stay safe for S3-backed storage.
- Add a durable backend path for adding existing images into an existing set and for creating a new set from selected existing images, not just from brand-new uploads.
- Add a durable backend path for moving a set from one subject to another, including storage-path updates, compatibility-layer updates, and any cascading references that depend on the set path.
- The backend and frontend now both preserve mixed existing tag IDs and newly typed tag names in DB-backed edit saves; browser-smoke one deployed tag edit after release to confirm the hosted frontend matches the local regression coverage.
- The hosted frontend already lowercases `subjectTags` and `setTags` before multipart upload and has direct test coverage for that request shape. The deployed backend upload-db relation-write fix has now passed a hosted browser smoke, so reopen frontend serialization questions only if a future upload regression reproduces empty upload-tag relations.
- Keep `gallery_v2.images` authoritative now that the live backfill is in place and `POST /api/delete-image-db` runs on it; future image-level edit routes should target catalog IDs rather than inferred filenames alone.
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