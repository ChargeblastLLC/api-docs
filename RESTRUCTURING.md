# API Docs Restructuring Plan

A living, maintainer-facing plan for cleaning up and organizing the Chargeblast API docs. This is **not published** to the site (it is a plain `.md` at the repo root and is not referenced in `docs.json`). Update the checklists here as work lands.

Tracking issue: [CHA-2109](https://linear.app/chargeblast/issue/CHA-2109/version-api-docs-navigation-collapse-duplicate-v2v3-endpoint-pages).

## Why

The docs grew organically and accumulated three problems:

1. **Duplicate-looking nav entries** for the same endpoint at different API versions (e.g. two "Fetch Alerts", two "Fetch Descriptors") listed side-by-side in one group.
2. **A dead second config** (`mint.json`) alongside the live `docs.json` — two navigation sources drifting apart.
3. **Orphaned pages** — `.mdx` files in the repo that no navigation references but that stay reachable by direct URL.

## Approach: strangler-fig, not big-bang

We do **not** rewrite the docs in one pass. We incrementally route each concern to the new structure and retire the old artifact only once its replacement is live, so every step is small and easy to revert. Each phase below is independently shippable.

## Target architecture

```
docs.json                      # single source of truth: flat groups nav, theme, redirects
openapi.json                   # one spec, all versions; single-version pages target ops via frontmatter
api-reference/<tag>/*.mdx       # one page per API operation (openapi: <method> <path>)
guides/*.mdx                    # task-oriented prose
guides/integrations/*.mdx       # per-processor setup guides
concepts/*.mdx                  # conceptual reference prose (overview, alert types, reason codes, lifecycle)
assets/logo/ assets/images/     # assets (single logo folder, single images tree)
favicon.png .gitignore
```

The repo does not match this yet; see [Repository structure](#repository-structure-target--cleanup) for the current-state gaps and the migration checklist. The `reference/` and split `logo/` + `images/` folders are being migrated to `concepts/` and `assets/`.

Rules:

- **One nav source.** Only `docs.json`. Never reintroduce `mint.json`.
- **Per-endpoint versioning via one OpenAPI-bound page per version; the current version owns the canonical slug, the legacy version is a deprecated sibling dropped from nav.** When an endpoint has multiple live API versions, keep each as its own `openapi:`-backed page. The current version lives at the clean slug (`fetch-alerts`) and stays in the nav; the legacy version lives at a `-vN` sibling (`fetch-alerts-v2`), is marked `deprecated: true`, dropped from the nav, and linked from the current page via a `<Note>`. The nav stays a single flat `groups` list with one entry per endpoint.
- **One file per operation.** Every endpoint page — current or legacy — is backed by `openapi.json` via `openapi: <method> <path>` frontmatter; prose lives in the page body (it renders above the auto-generated parameters/responses). This keeps the native color-coded method pill and interactive playground on every version.
- **Every removed URL gets a redirect.** Add a 308 entry to `docs.json` `redirects` whenever a page path goes away.

### Versioning decision (resolved)

**Versioning is applied per endpoint, not globally.** Each endpoint advances through API versions independently (alerts are at v2/v3, descriptors at v1/v2), so a single site-wide version axis would be misleading — selecting "v3" globally would imply a v3 exists for every endpoint. It therefore does not make sense to keep a global version dropdown, and Mintlify's `navigation.versions` dropdown was rejected for this reason.

**In-page `<Tabs>` switching was also tried and rejected.** A tabbed page must be hand-authored, because a Mintlify page cannot mix the auto-generated OpenAPI block with tabbed content. That costs the two things that make the other endpoint pages consistent: the native color-coded method pill (green `GET` / blue `POST`) in the sidebar and the interactive playground — both are generated only from `openapi:` frontmatter binding, and the `tag` frontmatter substitute renders a differently-styled badge on the wrong side. Consistency with the rest of the reference won out.

**Resolved approach:** keep every version as its own `openapi:`-bound page. The current version owns the canonical slug and the nav entry; the legacy version is a `deprecated: true` sibling (`-vN`), dropped from the nav but reachable by direct link and redirect, and surfaced from the current page with a `<Note>`. Reference implementation: `api-reference/alerts/fetch-alerts.mdx` (current, v3) + `api-reference/alerts/fetch-alerts-v2.mdx` (legacy, hidden).

## Repository structure (target + cleanup)

The repo layout drifted: `reference/` became a catch-all holding three live pages next to ~18 orphans (auto-named junk like `merchant-2`, `unenroll-2`, plus processor integration guides), `api-reference/getting-started/` mixes a live endpoint with orphaned prose, and assets are split across two logo folders. The goal is that a contributor can guess where any page lives from its purpose alone.

### Done (safe, no URL impact)

- [x] Removed tracked `.DS_Store` and added `.gitignore` (macOS + node/mintlify).

### Target folder taxonomy

| Folder | Holds | Naming |
| --- | --- | --- |
| `api-reference/<tag>/` | one page per OpenAPI operation; `<tag>` matches the OpenAPI tag | kebab-case verb-noun (`fetch-alerts`), no numeric suffixes |
| `guides/` | task-oriented walkthroughs | kebab-case |
| `guides/integrations/` | per-processor setup guides (Stripe, Klaviyo, …) | `<processor>` |
| `concepts/` (rename of the live part of `reference/`) | conceptual reference: overview, alert types, reason codes, lifecycle | kebab-case |
| `assets/logo/`, `assets/images/` | one logo folder, one images tree | descriptive |

Retire `reference/` as a dumping ground: its live pages move to `concepts/`, its orphans are triaged (below).

### Live-page moves (each needs a 308 redirect from the old path)

Do these as one reviewed batch; every move changes the public URL, so pair it with a `redirects` entry and re-run `mint broken-links`.

- [ ] `reference/welcome-to-chargeblast` -> `concepts/overview` (or keep slug, just move file and update `docs.json`)
- [ ] `reference/alert-types` -> `concepts/alert-types`
- [ ] `reference/reason-codes` -> `concepts/reason-codes`
- [ ] `api-reference/enrollment/enrollment-lifecycle` -> `concepts/enrollment-lifecycle` (it is prose, not an endpoint)
- [ ] `api-reference/getting-started/rdr-endpoint` -> `api-reference/rdr/rdr-endpoint` (drop the `getting-started` folder once emptied)

### Orphan reorg (folds into Phase 3 "retire an orphaned page")

For each, decide keep-and-move vs delete-and-redirect using the orphan playbook:

- [ ] Integration guides -> `guides/integrations/`: `stripe-integration`, `checkout-champ-integration`, `klaviyo-integration`, `mamopay-integration`, `slack-integration`, `zapier-integration` (verify external links first; these are likely reachable from the app/help center).
- [ ] `guides/digital-receipt-lookup`, `guides/tracker-snippet` -> add to nav under Deflections, or delete+redirect.
- [ ] `api-reference/getting-started/guide` -> reshape into an `api-reference/introduction/overview` page, or delete.
- [ ] `api-reference/getting-started/refund-endpoint`, `reference/refund-endpoint` -> reconcile the two refund pages into one; delete the loser with a redirect. (Also fixes the broken link flagged by `mint broken-links`.)
- [ ] `api-reference/webhooks/webhook-events` (`hidden: true`) -> either surface under the Webhooks group or delete.
- [ ] Junk duplicates to delete+redirect: `reference/{alerts-update, merchant-2, merchants-2, enroll-merchant-1, unenroll-2, orders-upload, refunding-alerts, bin-lookup, embedded, webhooks}`.

### Assets

- [ ] Consolidate the two logo folders (`logo/` and `images/logo/`) into one `assets/logo/`; update the `logo` field in `docs.json`.
- [ ] Remove currently-unreferenced assets after confirming they are unused: `logo/dark.svg`, `logo/light.svg`, `images/logo/logo-light-mode.png` (0 references today).
- [ ] Move `images/reference/*` screenshots alongside their owning section (or keep a single `assets/images/` tree) and update page `src` paths.

## Playbooks

### Collapse a duplicated endpoint pair into a current page + hidden legacy sibling

1. Identify the current page and the legacy page (by their `openapi:` frontmatter paths).
2. Put the **current** version at the endpoint's canonical slug (e.g. `api-reference/alerts/fetch-alerts`) with `openapi: <method> <current-path>` frontmatter. Add any callouts as body prose (they render above the auto-generated reference) plus a `<Note>` linking to the legacy page.
3. Put the **legacy** version at a `-vN` sibling slug (e.g. `api-reference/alerts/fetch-alerts-v2`) with `openapi: <method> <legacy-path>`, `deprecated: true`, and a `<Warning>` linking back to the current page.
4. In `docs.json`, list only the current page in the nav (the legacy sibling is intentionally omitted — it still builds and is reachable by URL). Add a 308 `redirects` entry from any retired slug (e.g. `/api-reference/alerts/fetch-alerts-v3`) to the canonical slug.
5. Run `mint broken-links` and check both pages render (with their native method pill + playground) via `mint dev`.

Reference implementation: `api-reference/alerts/fetch-alerts.mdx` (current, v3) + `api-reference/alerts/fetch-alerts-v2.mdx` (legacy, hidden).

### Retire an orphaned page

1. Confirm it is orphaned: not referenced in the `docs.json` nav (see script below).
2. `grep` the repo for internal links to its route; check whether the app or help center deep-links to it (orphans stay reachable by direct URL).
3. If the content is still valuable, **add it to the nav** instead of deleting. If superseded, delete the file and add a 308 `redirects` entry to its replacement.
4. Run `mint broken-links`.

### Consolidate stub endpoint pages into OpenAPI nav (future phase)

Per Mintlify's [Migrating MDX API pages to OpenAPI navigation](https://www.mintlify.com/docs/guides/migrating-from-mdx), an endpoint page that only carries `openapi: <method> <path>` frontmatter with no body does not need its own `.mdx` file. It can be referenced directly in a group's `pages` array as `"<METHOD> <path>"` with an `"openapi": "openapi.json"` field on the group, and the stub file deleted. This reduces the number of files to maintain and keeps endpoint pages consistent.

Do this as its own reviewed batch, not bundled with a versioning change, because of two caveats:

- **Pages with a body stay MDX** (or move their prose into the spec via the `x-mint` extension): anything with callouts or prose above the auto-generated reference. The versioned endpoint pages (`fetch-alerts` + `fetch-alerts-v2`, `fetch-descriptors` + `fetch-descriptors-v1`) are `openapi:`-bound MDX pages carrying a `<Note>`/`<Warning>` body.
- **URL stability**: a directly-referenced operation generates its own slug, which differs from the current file-path slug (e.g. `api-reference/sync-data/track`). Add a 308 `redirects` entry for every changed URL.

Zero-body stub candidates (safe first batch — verify each still has an empty body first):

- [ ] `api-reference/credit-requests/create`
- [ ] `api-reference/deflections/logs`
- [ ] `api-reference/enrollment/enroll-merchant`
- [ ] `api-reference/enrollment/enrollment-status`
- [ ] `api-reference/enrollment/fetch-merchant`
- [ ] `api-reference/enrollment/unenroll-merchant`
- [ ] `api-reference/sync-data/get-order`
- [ ] `api-reference/sync-data/get-orders`
- [ ] `api-reference/sync-data/track`
- [ ] `api-reference/sync-data/upload-orders`

Near-empty pages to evaluate (move body to `x-mint` or keep): `fetch-an-alert`, `update-alert`, `fetch-merchants`.

### Find orphaned pages

```bash
python3 - <<'EOF'
import json, glob
d = json.load(open('docs.json'))
refd = set()
def walk(o):
    if isinstance(o, dict):
        for k, v in o.items():
            if k == 'pages':
                for p in v:
                    walk(p) if not isinstance(p, str) else refd.add(p)
            else: walk(v)
    elif isinstance(o, list):
        for i in o: walk(i)
walk(d['navigation'])
for p in sorted(x[:-4] for x in glob.glob('**/*.mdx', recursive=True)):
    if p not in refd: print(p)
EOF
```

## Progress

### Done

- [x] Alerts versioning — current v3 owns `api-reference/alerts/fetch-alerts` (`openapi: get /api/v3/alerts`); legacy v2 moved to hidden, deprecated `api-reference/alerts/fetch-alerts-v2`; 308 redirect from `fetch-alerts-v3` to the canonical slug.
- [x] Descriptors versioning — current v2 owns `api-reference/enrollment/fetch-descriptors` (`openapi: get /api/v2/descriptors`); legacy v1 moved to hidden, deprecated `api-reference/enrollment/fetch-descriptors-v1`; 308 redirect from `fetch-descriptors-v2` to the canonical slug.
- [x] Nav — flat `groups` list (single entry per endpoint).
- [x] Retired the earlier global `navigation.versions` dropdown and the in-page `<Tabs>` experiment in favour of per-version `openapi:`-bound pages (current in nav, legacy hidden).
- [x] Deleted `mint.json`.
- [x] Orphan cleanup (batch 1) — deleted `reference/alerts-5`, `reference/alert-1`, `reference/descriptors-2`, `api-reference/alerts/alert`, each with a redirect.
- [x] Removed tracked `.DS_Store` and added `.gitignore`.

### Remaining orphans (per-batch review)

Verify links before removing each; keep-and-nav if still valuable, else delete + redirect.

- [ ] `reference/welcome-to-chargeblast-copy` — distinct "Getting Set Up" onboarding content; decide keep-and-nav vs merge into `reference/welcome-to-chargeblast`.
- [ ] `reference/alerts-update`
- [ ] `reference/merchant-2`, `reference/merchants-2`
- [ ] `reference/enroll-merchant-1`, `reference/unenroll-2`
- [ ] `reference/orders-upload`
- [ ] `reference/refund-endpoint`, `reference/refunding-alerts`
- [ ] `reference/bin-lookup`
- [ ] `reference/embedded`
- [ ] `reference/webhooks`
- [ ] Integration pages (likely externally linked — verify first): `reference/stripe-integration`, `reference/checkout-champ-integration`, `reference/klaviyo-integration`, `reference/mamopay-integration`, `reference/slack-integration`, `reference/zapier-integration`
- [ ] `api-reference/getting-started/guide`
- [ ] `api-reference/getting-started/refund-endpoint` — also has a pre-existing broken link to `/reference/sync-data/upload-orders`; fix or remove.
- [ ] `api-reference/webhooks/webhook-events`
- [ ] `guides/digital-receipt-lookup`, `guides/tracker-snippet`

### Known follow-ups

- [ ] Fix the one broken link surfaced by `mint broken-links` (in `api-reference/getting-started/refund-endpoint`).
- [ ] Consolidate zero-body endpoint stubs into direct OpenAPI nav references (see the playbook above).
