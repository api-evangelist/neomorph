---
name: neomorph-track-company-news
description: >-
  Pull Neomorph's press releases and media coverage from its content API, newest first, with the
  category each item belongs to. Use this to answer "what has Neomorph announced", to watch for new
  financings, partnerships, clinical milestones or appointments, or to build a dated timeline of the
  company.
generated: '2026-08-26'
method: generated
source: >-
  Grounded in openapi/neomorph-news-api-openapi.yml and openapi/neomorph-taxonomy-api-openapi.yml in
  this repository, both derived from live anonymous probes of https://neomorph.com/wp-json on
  2026-08-26. Every operationId below appears in those documents.
api: neomorph:neomorph-news-api
base_url: https://neomorph.com/wp-json
operations:
  - listPosts
  - getPosts
  - listCategories
---

# Track Neomorph company news

Neomorph publishes no press API. This works against the WordPress content API behind its corporate
site. It is read-only, needs no credential, and is unsupported by the company — treat it as best
effort and cache what you get.

## Steps

1. **Resolve the categories once.** `GET /wp/v2/categories` (`listCategories`). On 2026-08-26 exactly
   two terms exist: `Press Release` (id 1, 13 posts) and `In The Media` (id 8, 1 post). Hold the
   id→name map; you will need it because posts carry category ids, not names.

2. **List the news, newest first.** `GET /wp/v2/posts` (`listPosts`) with
   `?orderby=date&order=desc&per_page=100&_fields=id,date,modified,slug,title,link,categories,excerpt`.
   `per_page` is capped at 100 — exceeding it returns HTTP 400 `rest_invalid_param` with
   `data.details.per_page.code = rest_out_of_bounds`. The whole archive was 14 items at harvest, so
   one page is enough today; still read `X-WP-Total` rather than assuming.

3. **Page only if you must.** Read `X-WP-Total` and `X-WP-TotalPages` from the response headers, or
   follow the RFC 8288 `Link` header's `rel="next"`. Do not construct page URLs by hand.

4. **Fetch a single item when you need the body.** `GET /wp/v2/posts/{id}` (`getPosts`). `content.rendered`
   is HTML, not markdown or plain text — strip it before feeding it to a model.

5. **Watch for new items incrementally.** Re-run step 2 with `?after=<ISO 8601 timestamp of your last
   run>` to get only what is new. Use `modified_after` instead if you also care about edits to
   existing releases.

## Interpreting what you get

- The archive is small and slow-moving: 14 items spanning 2020-12-20 to 2026-05-13. An empty delta is
  the normal result, not a failure.
- `excerpt.rendered` is usually the release's opening paragraph and is often enough on its own.
- `author` is an integer id you cannot resolve — `/wp/v2/users` is blocked at the edge with an HTML
  403. Do not try; do not report the id as a person.
- `featured_media` is a media id. Resolve it with `GET /wp/v2/media/{id}` or, cheaper, add `?_embed`
  to step 2 and read `_embedded['wp:featuredmedia']`.

## Rules

- No rate limits are published and no rate-limit headers are returned. There is nothing to back off
  on, so be conservative: one pass per run, `_fields` always set, and cache between runs.
- Errors are the WordPress `WP_Error` envelope served as `application/json` — `{"code":..., "message":...,
  "data":{"status":...}}`. Branch on `code`, never on `message`. See
  `errors/neomorph-problem-types.yml`.
- Every operation here is a GET. There is nothing to make idempotent and nothing to reverse. Do not
  attempt any write — the write routes exist but require a WordPress credential Neomorph does not
  issue to third parties.
- Do not present this data as coming from a Neomorph API product. It is the CMS behind the marketing
  site, and it can change or disappear on a plugin update without notice.
