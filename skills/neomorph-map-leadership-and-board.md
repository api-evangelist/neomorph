---
name: neomorph-map-leadership-and-board
description: >-
  Enumerate Neomorph's people and how the company groups them — Management, Board of Directors,
  Scientific Founders, Scientific Advisory Board. Use this to answer "who runs Neomorph", "who is on
  the board", or "who are the scientific founders". Read the limitations section first: titles and
  biographies are NOT available from this API.
generated: '2026-08-26'
method: generated
source: >-
  Grounded in openapi/neomorph-team-api-openapi.yml in this repository, derived from live anonymous
  probes of https://neomorph.com/wp-json on 2026-08-26.
api: neomorph:neomorph-team-api
base_url: https://neomorph.com/wp-json
operations:
  - listTeamCategory
  - listTeam
  - getTeam
---

# Map Neomorph's leadership and board

## Steps

1. **Get the groupings.** `GET /wp/v2/team_category` (`listTeamCategory`) with
   `?per_page=100&_fields=id,name,slug,count`. Four terms at harvest:

   | id | name | count |
   |----|------|-------|
   | 2 | Management | 11 |
   | 3 | Board of Directors | 7 |
   | 4 | Scientific Founders | 4 |
   | 10 | Scientific Advisory Board | 0 |

   `count` is authoritative and worth reporting. Scientific Advisory Board is **registered but
   empty** — the term exists in the CMS and no person is assigned to it. Do not report an SAB from
   the existence of the category.

2. **List the people.** `GET /wp/v2/team` (`listTeam`) with
   `?per_page=100&_fields=id,slug,title,link,team_category,featured_media,class_list`. 19 published
   records at harvest.

3. **Group them.** Each record's `team_category` is an array of term ids from step 1. A person can
   appear in more than one group. `class_list` also carries a readable slug —
   `team_category-board-of-directors` — if you would rather not join on ids.

4. **Filter server-side when you only want one group.** `GET /wp/v2/team?team_category=3` returns just
   the board.

5. **Get a portrait.** `featured_media` is a media id; resolve with `GET /wp/v2/media/{id}` and read
   `source_url`, or add `?_embed` in step 2 and read `_embedded['wp:featuredmedia'][0].source_url`.

## Limitation you must respect

**Job titles and biographies are not in this API.** The `team` post type has **no `content` field at
all**, and its `acf` object returns an **empty array** to unauthenticated callers — verified on team
id 1535 on 2026-08-26. What you can get is: name (`title.rendered`), slug, permalink, portrait, and
group membership.

If you need a title or a bio, follow `link` and read the HTML page — and say that is where it came
from. **Never infer a person's role from their team_category alone** ("Board of Directors" is not a
title), and never generate a biography.

## Rules

- Read-only, anonymous. No rate limits published; set `_fields` and cache.
- These records are named individuals. Report only what the API returns, attribute it to
  neomorph.com, and do not enrich it against other sources inside this skill.
- Branch on the `code` field of the `WP_Error` envelope, not on `message`.
