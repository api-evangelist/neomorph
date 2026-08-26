---
name: neomorph-search-the-site
description: >-
  Search everything Neomorph publishes — news, corporate pages, team members and publications — in one
  call, then resolve any hit to its full record. Use this as the entry point when you do not yet know
  which collection holds the answer.
generated: '2026-08-26'
method: generated
source: >-
  Grounded in openapi/neomorph-search-api-openapi.yml and openapi/neomorph-discovery-api-openapi.yml in
  this repository, derived from live anonymous probes of https://neomorph.com/wp-json on 2026-08-26.
api: neomorph:neomorph-search-api
base_url: https://neomorph.com/wp-json
operations:
  - search
  - getIndex
  - listTypes
  - getPosts
  - getPages
  - getTeam
  - getResource
---

# Search everything Neomorph publishes

## Steps

1. **Search.** `GET /wp/v2/search` (`search`) with `?search=<terms>&per_page=100`. Returns a compact
   record per hit: `id`, `title`, `url`, `type`, `subtype`. A search for `neomorph` returned
   `X-WP-Total: 32` on 2026-08-26 — which is roughly the whole site, so use specific terms.

2. **Read the total before paging.** `X-WP-Total` and `X-WP-TotalPages` are response headers and are
   exposed cross-origin. `per_page` maxes at 100.

3. **Route each hit by `subtype`.** This is the field that tells you which collection to go back to:

   | subtype | resolve with | operationId |
   |---------|--------------|-------------|
   | `post` | `GET /wp/v2/posts/{id}` | `getPosts` |
   | `page` | `GET /wp/v2/pages/{id}` | `getPages` |
   | `team` | `GET /wp/v2/team/{id}` | `getTeam` |
   | `resource` | `GET /wp/v2/resource/{id}` | `getResource` |

   The full `subtype` enum the route declares is `post`, `page`, `e-floating-buttons`,
   `elementor_library`, `team`, `resource`, `astra-advanced-hook`, `category`, `post_tag`,
   `team_category`, `any`. Confirm the live set rather than hard-coding it: `GET /wp/v2/types`
   (`listTypes`) lists every registered post type and its `rest_base`, and `GET /` (`getIndex`)
   returns the deployment's whole 432-route index.

   **Dangling hits.** Search does not filter to what you can then fetch. A search for `neomorph` on
   2026-08-26 returned 32 hits: 14 `post`, 12 `page` and **6 `elementor_library`** — and
   `/wp/v2/elementor_library` is auth-gated, returning HTTP 401 `rest_forbidden`. Those six are page
   templates, not content. Drop any hit whose subtype you cannot resolve, and do not report its title
   as a Neomorph publication; it is a fragment of the site's page builder.

4. **Narrow at the source when you can.** `type` and `subtype` are accepted as query parameters on
   `/wp/v2/search`, so `?search=NEO-811&subtype=post` beats filtering client-side.

## Interpreting what you get

- `url` is a live permalink; it is safe to cite.
- Search covers only **publicly queryable** types. Media is not searchable this way — list
  `/wp/v2/media` directly.
- Relevance ordering is WordPress's, not tuned. For a small site like this one, listing the relevant
  collection outright is often better than searching it.

## Rules

- Read-only, anonymous, no credential.
- On HTTP 400 read `data.details.<param>.code` for the actual reason; do not retry unchanged.
- No rate limits are published and no rate-limit headers are returned — one pass per run, and cache.
- This is the search index of a corporate website, not a product API. Say so when you cite it.
