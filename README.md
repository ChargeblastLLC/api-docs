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

- `docs.json` — the single source of truth for navigation, theme, versions, and redirects. (The old `mint.json` schema is no longer used; do not reintroduce it.)
- `openapi.json` — one OpenAPI spec covering every version. Endpoint pages target an operation with `openapi: <method> <path>` frontmatter (e.g. `get /api/v3/alerts`).
- `api-reference/<tag>/*.mdx` — one page per API operation, backed by `openapi.json`.
- `guides/*.mdx` — task-oriented prose (implementation walkthroughs).
- `reference/*.mdx` — conceptual reference prose (alert types, reason codes, welcome).
- `images/`, `logo/`, `favicon.png` — assets.

## Versioning

Parallel API versions live under `navigation.versions` in `docs.json`, rendered as a version dropdown in the sidebar. Each version has its own group tree; shared (non-versioned) pages are listed in both trees by referencing the same `.mdx` file — no content is duplicated.

- `v3` is the default (`"default": true`).
- Only the pages that actually differ by version are swapped between trees. Today that is:
  - Alerts: `api-reference/alerts/fetch-alerts-v3` (v3) vs `api-reference/alerts/fetch-alerts` (v2, `GET /api/v2/alerts`).
  - Enrollment: `fetch-descriptors-v2` (v3) vs `fetch-descriptors` (v2, `GET /api/descriptors`).

When a new API version of an existing endpoint ships:

1. Add the new endpoint `.mdx` under `api-reference/<tag>/`.
2. Add a new entry to `navigation.versions` and set `"default": true` on it (removing it from the previous default).
3. Add a deprecation `<Warning>` to the superseded page pointing readers to the new version.

Do not add `-vN` sibling pages inside the same nav group — that is exactly the duplication the version dropdown replaces.

## Cleanup status (strangler-fig)

Navigation was migrated from a flat `groups` list to versions without moving or renaming any live URLs, and legacy artifacts are being retired incrementally. The full plan, per-endpoint playbooks, progress checklist, and remaining orphan inventory live in [RESTRUCTURING.md](RESTRUCTURING.md) (tracked by [CHA-2109](https://linear.app/chargeblast/issue/CHA-2109/version-api-docs-navigation-collapse-duplicate-v2v3-endpoint-pages)). Read it before adding or removing pages.
