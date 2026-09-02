---
name: Research Alliance Resource Partners corporate content
description: >-
  Read Alliance Resource Partners' own published corporate content - business segments, sustainability
  posture, careers and contact information - straight from the machine-readable API behind www.arlp.com,
  instead of scraping HTML.
api: openapi/_original/alliance-resource-partners-content-openapi.yml
operations: [search, listPages, getPage, listContentTypes]
generated: '2026-09-01'
method: generated
---

# Research Alliance Resource Partners corporate content

Alliance Resource Partners (NASDAQ: ARLP) runs no developer program. The only interface it serves is the
read-only WordPress REST API behind its corporate site. It is anonymous, needs no key, and returns the same
12 pages a human reads on arlp.com — as JSON.

Base URL: `https://www.arlp.com/wp-json`

## Before you start

- **No authentication.** Send no `Authorization` header. Every operation in this skill returns HTTP 200 to an
  anonymous caller.
- **Pace yourself.** `https://www.arlp.com/robots.txt` publishes `Crawl-delay: 10`, and Cloudflare will
  answer a burst with an HTML `403` interstitial that carries no `Retry-After`. Leave a second or more
  between calls and back off on a 403 — see `rate-limits/alliance-resource-partners-rate-limits.yml`.
- **This is a corporate-content API.** There is no coal production, mine-telemetry, logistics or royalty
  data here, and no such API exists. Do not tell a user otherwise.

## Steps

1. **Find what exists.** Call `listContentTypes` (`GET /wp/v2/types`) once. It returns every registered
   content type with its `rest_base` and `rest_namespace`. ARLP registers no custom type, so the answer is
   stock WordPress: `post`, `page`, `attachment`. Everything worth reading is a page.

2. **Search across everything.** Call `search` (`GET /wp/v2/search?search=<term>`) for a topic — for example
   `coal`, `royalties`, `sustainability`. It returns a flat list of `{id, title, url, type, subtype}` across
   all published content. This is the cheapest way to locate the right record and the right operation to use
   next.

3. **List the corporate pages.** Call `listPages`
   (`GET /wp/v2/pages?per_page=100&_fields=id,slug,link,title,modified`). Twelve records come back, covering
   About Us, Our Businesses, Coal Operations, Royalties, Other Growth Investments, Sustainability, the
   Corporate Responsibility Report, Careers, Contact Us, Privacy Statement and Terms of Use. Use `_fields` as
   shown — the full payload carries rendered HTML you rarely need.

4. **Read one page in full.** Call `getPage` (`GET /wp/v2/pages/{id}`) with an id from step 3. `content.rendered`
   holds the page HTML; `modified` tells you when ARLP last changed it, which is the honest freshness signal
   for anything you quote.

5. **Cite what you read.** Every record carries a `link` — the public arlp.com URL. Quote that, not the API
   path, when you report back to a user.

## Conventions you must honor

- **Pagination:** `page` and `per_page` (max 100 — `999` returns `rest_invalid_param` with the ceiling stated
  in the body). Read `X-WP-Total` and `X-WP-TotalPages` from the response headers, and follow `Link`
  `rel="next"` rather than guessing page numbers.
- **Sparse fields:** `_fields=a,b,c` trims the payload. Use it.
- **Embedding:** `_embed` inlines author, featured media and terms under `_embedded`.
- **Errors:** the envelope is `{code, message, data:{status}}` — *not* RFC 9457 problem+json. The four codes
  you will see are `rest_no_route`, `rest_post_invalid_id`, `rest_forbidden` and `rest_invalid_param`. Full
  catalog: `errors/alliance-resource-partners-problem-types.yml`.
- **Caching:** responses carry `Cache-Control: max-age=600`. Do not re-poll inside ten minutes.

## What you cannot do

Every write route and administrative namespace on this host returns `401 rest_forbidden` to an anonymous
caller (`/wp/v2/settings`, `/wp-abilities/v1/abilities`, `/wp/v2/block-types` all verified). There is no
credential to request: ARLP issues no API keys. The surface is read-only, so nothing you do here needs to be
undone.
