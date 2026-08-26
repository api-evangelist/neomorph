---
name: neomorph-read-publications
description: >-
  Retrieve the peer-reviewed molecular glue and targeted protein degradation literature Neomorph
  surfaces on its Publications page. Use this to build the scientific bibliography behind the company's
  platform, or to check what the founding team has published.
generated: '2026-08-26'
method: generated
source: >-
  Grounded in openapi/neomorph-publications-api-openapi.yml in this repository, derived from live
  anonymous probes of https://neomorph.com/wp-json on 2026-08-26.
api: neomorph:neomorph-publications-api
base_url: https://neomorph.com/wp-json
operations:
  - listResource
  - getResource
---

# Read Neomorph's publications

## The one thing that trips people up

The collection is called **Publications** on the website, but its `rest_base` is `resource`. Address
`/wp/v2/resource`. There is no `/wp/v2/publications` — that path returns HTTP 404 `rest_no_route`.

## Steps

1. **List the publications.** `GET /wp/v2/resource` (`listResource`) with
   `?orderby=date&order=desc&per_page=100&_fields=id,date,slug,title,link,content,menu_order`.
   Eleven records at harvest, spanning 2014-01-17 to 2022-11-03.

2. **Fetch one.** `GET /wp/v2/resource/{id}` (`getResource`).

3. **Order by editorial intent, not date, when it matters.** `menu_order` is the site's own manual
   ordering field. `?orderby=menu_order&order=asc` reproduces the sequence the Publications page shows.

## Known gap — read this before you promise a citation

The API returns the publication **title**, **slug**, **link** and a `content.rendered` HTML blob. It
does **not** return structured citation metadata. Journal, DOI, year of publication, and author list
are held in Advanced Custom Fields, and the `acf` object reads as an **empty array** for
unauthenticated callers — verified on resource id 1430 on 2026-08-26.

So:

- Parse `content.rendered` for citation text if it is there, and say that you did.
- Or follow `link` and read the HTML page.
- Do **not** synthesise a DOI, a journal name or an author list. If you cannot read it, report that it
  is not exposed. See `data-model/neomorph-data-model.yml` for the full account of this gap.

Note also that `date` is the WordPress post date — when the entry was added to the site — and is not
reliably the paper's publication date. Several entries were backdated to approximate it, but nothing
in the API states which.

## Rules

- Read-only, anonymous, no credential. No rate limits published; set `_fields` and cache.
- Branch on the `code` field of the `WP_Error` envelope, not on `message`.
- These are third-party papers in journals Neomorph does not own. Cite the journal, not this API.
