---
name: ad-library-weekly-diff
description: Track what competitors are advertising across the Meta, Google, LinkedIn and TikTok public ad libraries and report what changed since last week — new ads, ads that stopped running, and new creative angles. Uses the Captapi API (MCP tools or REST with CAPTAPI_API_KEY). Shows a credit estimate before running, keeps a snapshot file between runs, and cites the endpoint behind every claim. Use when someone asks to monitor competitor ads, compare ad activity week over week, or audit a brand's paid creative.
---

# Ad library weekly diff

Pull each competitor's currently visible ads from the public ad libraries,
compare them with last week's snapshot, and write a short report of what is
new, what stopped, and what the new ads are saying.

## What public ad libraries can and cannot tell you

State these limits in every report. Do not let the reader infer more.

- Public ad libraries do **not** show spend, impressions or conversions for
  ordinary commercial ads. Google publishes spend and impression ranges only
  for election ads; Meta shows reach breakdowns only for some EU-delivered and
  political ads; LinkedIn shows impressions and targeting on a subset of ads.
  Never estimate budget or performance from ad counts.
- "Not in the library" is not "not advertising": libraries are country-scoped
  and can lag by a day or more.
- TikTok's ad library only covers ads shown in the EU/EEA, the UK and
  Switzerland. Outside those countries, say "TikTok: not covered" — do not
  report zero ads.
- An ad's first/last shown dates are calendar days, not exact times.

## Inputs to collect first

1. Competitors: brand name, and if known their Meta page URL, Google advertiser
   name and LinkedIn company name.
2. Countries (two-letter codes). Default: `US`. Add an EU/UK country such as
   `GB` or `DE` if TikTok matters.
3. Snapshot file path. Default: `ad-library-snapshot.json` in the working
   directory.
4. Whether to pull ad details and copy for new ads (more credits, better report).

## Credit estimate — show before running

Per competitor, per country:

| Step | Endpoint | Credits |
|---|---|---|
| Find Meta page | `/v1/ad-library/facebook/search-companies` | 2 |
| Meta ads | `/v1/ad-library/facebook/company-ads` | 2 |
| Find Google advertiser | `/v1/ad-library/google/advertiser-search` | 1 |
| Google ads | `/v1/ad-library/google/company-ads` | 2 |
| LinkedIn ads | `/v1/ad-library/linkedin/search-ads` | 2 |
| TikTok ads (EU/EEA/UK/CH only) | `/v1/ad-library/tiktok/search` | 2 (up to 5 on its fallback path) |

That is about **11 credits** per competitor per country (9 where TikTok is not
covered). The lookup steps can be skipped on later runs by saving the page ID
and advertiser ID in the snapshot.

Optional, per **new** ad only: details (`.../ad-details`, 2 each) and Meta ad
copy (`/v1/ad-library/facebook/ad-transcript`, 2 each). Cap these — for
example at 5 new ads per competitor — and say so in the estimate.

Print the estimate (competitors × countries × 11, plus the detail cap) and
check the balance with `GET /v1/account/balance` (free). Ask the user to
confirm before spending.

## How to call Captapi

Either path works; use whichever the environment has.

- **MCP:** tools named after the endpoint, for example
  `facebook_ad_library_company_ads`, `google_ad_library_company_ads`,
  `linkedin_ad_library_search_ads`, `tiktok_ad_library_search`.
- **REST:** `GET https://api.captapi.com<path>?<params>` with header
  `Authorization: Bearer $CAPTAPI_API_KEY`.

Failed calls cost 0 credits. A `retryable: true` error can be retried once
after a short wait; `retryable: false` is a final answer — record it, do not
loop.

## Steps

1. **Load the snapshot** if it exists. It maps
   `competitor → country → library → {ids: [...], meta: {...}}` plus saved
   lookup IDs.
2. **Resolve IDs** only where the snapshot has none:
   - Meta: `facebook/search-companies?q=<brand>&country=<cc>` → pick the
     name-matched page; keep its `pageId`. If several look right, ask.
   - Google: `google/advertiser-search?q=<brand>&country=<cc>` → keep the
     advertiser ID of the matching result.
3. **Pull current ads** for each competitor and country:
   - `facebook/company-ads?url=<pageId or page URL>&country=<cc>&status=active`
   - `google/company-ads?advertiser=<advertiser id or name>&country=<cc>`
   - `linkedin/search-ads?company=<name>&country=<cc>`
   - `tiktok/search?q=<brand>&country=<cc>` — only for EU/EEA/UK/CH.
4. **Diff by ad ID** against the snapshot: `new` (in now, not before),
   `stopped` (before, not now), `continuing`.
5. **Enrich new ads** up to the agreed cap: `.../ad-details` for dates and
   media, `facebook/ad-transcript` for Meta headline/body/CTA/landing URL.
6. **Write the report** (format below) and **save the new snapshot** — only
   after the report is written, so a failed run never overwrites last week.

## Report format

For each competitor:

- One line per library: `new N · stopped N · running N`, or the reason it is
  missing (`not covered in this country`, `lookup failed`, `0 ads found`).
- **New ads:** headline or first line of copy, format (image/video/carousel),
  first shown date, landing domain, and the endpoint it came from, e.g.
  `(source: /v1/ad-library/facebook/company-ads)`.
- **Themes:** 2–4 bullets on angles the new ads share (offer, audience,
  product). Mark each as `[needs human review]` — it is an interpretation.
- **Stopped ads:** count, plus the longest-running one that stopped.

End with the limits section above, a one-line credit total for the run, and
the snapshot path.

## Where a human must review

- Which page or advertiser is "the" competitor when the lookup returns several.
- Every theme or strategy claim — the libraries show creatives, not intent.
- Any conclusion about budget or performance: there is no data for it; the
  report must say so rather than guess.
