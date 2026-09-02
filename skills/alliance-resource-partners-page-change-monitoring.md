---
name: Monitor Alliance Resource Partners page changes
description: >-
  Detect when Alliance Resource Partners edits a corporate page - business segments, sustainability posture,
  careers, terms - by polling the modified timestamps on its own content API rather than diffing HTML.
api: openapi/_original/alliance-resource-partners-content-openapi.yml
operations: [listPages, getPage, listPosts]
generated: '2026-09-01'
method: generated
---

# Monitor Alliance Resource Partners page changes

ARLP publishes no changelog, no status page and no RSS-driven newsroom on www.arlp.com. But its content API
exposes a `modified` timestamp per record and supports `modified_after` filtering, which is enough to watch
the corporate site for edits precisely.

Base URL: `https://www.arlp.com/wp-json`

## Steps

1. **Take a baseline.** Call `listPages`
   (`GET /wp/v2/pages?per_page=100&_fields=id,slug,link,title,modified`) and store `{id, modified}` for all
   12 records. There is exactly one page of results — `X-WP-TotalPages` confirms it.

2. **Poll for edits.** On each subsequent run call
   `GET /wp/v2/pages?modified_after=<last-run ISO 8601>&per_page=100&_fields=id,slug,link,title,modified`.
   An empty array means nothing changed. `modified_after` is a real parameter on this route — it comes from
   the site's own argument schema in the live discovery document, not from an assumption.

3. **Fetch the changed record.** For each returned id call `getPage` (`GET /wp/v2/pages/{id}`) and diff
   `content.rendered` against your stored copy.

4. **Cover posts too, if you care.** `listPosts` (`GET /wp/v2/posts?modified_after=...`) takes the same
   parameter. Be aware the ARLP blog holds a single 2023 post and its comment thread is spam — treat activity
   there as noise, not news.

5. **Do not confuse this with news.** ARLP's press releases and SEC filings live on `investor.arlp.com`, a
   Q4 Inc-hosted investor-relations platform that is *not* covered by this API and that answers automated
   requests with an HTTP 403 bot challenge. If a user asks for earnings or filings, send them to
   `https://investor.arlp.com/investors/press-releases/default.aspx` or to SEC EDGAR — do not claim this API
   carries them.

## Polling budget

`robots.txt` publishes `Crawl-delay: 10` and responses carry `Cache-Control: max-age=600`. Once per hour is
generous for a 12-page corporate site; once per day is honest. A tight loop earns a Cloudflare `403` with no
`Retry-After`.

## Safety

Read-only. Every operation is a `GET`, every write route returns `401 rest_forbidden`, and there is nothing
on this surface to reverse.
