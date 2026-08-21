# Chargeblast API Docs

Mintlify documentation for the Chargeblast API, published at [docs.chargeblast.com](https://docs.chargeblast.com).

## Development

Install the [Mintlify CLI](https://www.npmjs.com/package/mint) and preview locally:

```
npm i -g mint
mint dev
```

Before opening a PR, check for broken links:

```
mint broken-links
```

Changes deploy to production automatically once merged to the default branch (via the Mintlify GitHub App).

## Layout

- `docs.json` — the single source of truth for navigation (a flat `groups` list), theme, and redirects. (The old `mint.json` schema is no longer used; do not reintroduce it.)
- `openapi.json` — one OpenAPI spec covering every version. Auto-generated endpoint pages target an operation with `openapi: <method> <path>` frontmatter (e.g. `get /api/v2/alerts`). Multi-version endpoints are hand-authored instead (see Versioning).
- `api-reference/<tag>/*.mdx` — one page per API operation, backed by `openapi.json`.
- `guides/*.mdx` — task-oriented prose (implementation walkthroughs).
- `reference/*.mdx` — conceptual reference prose (alert types, reason codes, welcome).
- `images/`, `logo/`, `favicon.png` — assets.

## Versioning (per-endpoint, in-page tabs)

When an endpoint has multiple live API versions, document them on a **single page** with a `<Tabs>` switcher — one `<Tab>` per version — rather than as separate sibling pages or a global site version dropdown. The user switches versions inside the endpoint page.

- Example: [`api-reference/alerts/fetch-alerts.mdx`](api-reference/alerts/fetch-alerts.mdx) has a `v3 (current)` tab and a `v2` tab.
- The nav (`docs.json`) is a single flat `groups` list with **one** entry per endpoint (`api-reference/alerts/fetch-alerts`).
- Old per-version URLs (e.g. `/api-reference/alerts/fetch-alerts-v3`) 308-redirect to the merged page.

Tradeoff to know: because a Mintlify page is either an auto-generated OpenAPI endpoint (`openapi:` frontmatter) **or** regular MDX, a tabbed multi-version page cannot embed the auto-generated playground/schema. Tab contents are hand-authored with `<ParamField>` / `<ResponseField>` / `<RequestExample>` / `<ResponseExample>`, so keep them in sync with `openapi.json` by hand.

When a new API version of an existing endpoint ships:

1. Add a new `<Tab title="vN (current)">` to the endpoint page and demote the previous tab.
2. Keep a single nav entry; do not add a `-vN` sibling page.
3. Add a `<Warning>` in the legacy tab pointing to the current one.

Do not reintroduce `navigation.versions` (a global, whole-site dropdown) for per-endpoint versioning, and do not add `-vN` sibling pages in the nav.

## Cleanup status (strangler-fig)

Duplicate per-version endpoint pages were collapsed into single tabbed pages, and legacy artifacts are being retired incrementally. The full plan, per-endpoint playbooks, progress checklist, and remaining orphan inventory live in [RESTRUCTURING.md](RESTRUCTURING.md) (tracked by [CHA-2109](https://linear.app/chargeblast/issue/CHA-2109/version-api-docs-navigation-collapse-duplicate-v2v3-endpoint-pages)). Read it before adding or removing pages.
