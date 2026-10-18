---
name: competitor-content-digest
description: Produce a weekly digest of what competitors published on TikTok, Instagram, YouTube, X, Threads, LinkedIn and Facebook — their latest posts, which ones outperformed, and the topics and formats they are leaning into. Uses the Captapi API (MCP tools or REST with CAPTAPI_API_KEY). Shows a credit estimate first and cites the endpoint behind every claim. Use when someone asks to monitor competitor social content, benchmark posting activity, or find what is working for rival brands.
---

# Competitor content digest

Pull each competitor's latest public posts on the platforms they use, rank
them by engagement relative to that account's own baseline, and summarise
what changed in their content this period.

## What this can and cannot tell you

- Public, logged-out data only. No reach, saves, click-through or audience
  data — never estimate them.
- Platforms differ in what a listing returns:
  - X `user-tweets` returns the account's **most popular** recent tweets
    (around 100), not a chronological feed.
  - LinkedIn `company-posts` returns the posts a company page shows
    logged-out visitors — about the 10 latest.
  - Facebook `profile-posts` usually returns only **1–2** of the latest posts
    a page shows logged-out visitors. Treat Facebook as a spot check, not a
    full feed, and say so in the digest.
- Compare engagement within one account and one platform. Raw likes across
  platforms are not comparable.

## Inputs to collect first

1. Competitors and their profile URLs per platform. Do not guess handles —
   skip a platform the user did not provide.
2. Period (default: last 7 days) and posts to pull per platform (default 10;
   Facebook `profile-posts` accepts at most `limit=4`).
3. Previous digest file, if any, to report week-over-week change.

## Credit estimate — show before running

Per competitor, one listing call per platform:

| Platform | Endpoint | Credits |
|---|---|---|
| TikTok | `/v1/tiktok/channel-posts` | 2 |
| Instagram | `/v1/instagram/channel-posts` (0.3 per post; 10 posts) | 3 |
| YouTube | `/v1/youtube/channel-videos` | 2 |
| X | `/v1/twitter/user-tweets` | 2 |
| Threads | `/v1/threads/user-posts` | 2 |
| LinkedIn | `/v1/linkedin/company-posts` | 1 |
| Facebook | `/v1/facebook/profile-posts` | 2 |

All seven platforms: about **14 credits** per competitor; ~16 with a couple of
optional detail calls. Print the total for the requested competitors and
platforms, check `GET /v1/account/balance` (free), and ask to confirm.

## How to call Captapi

- **MCP:** `tiktok_channel_posts`, `instagram_channel_posts`,
  `youtube_channel_videos`, `twitter_user_tweets`, `threads_user_posts`,
  `linkedin_company_posts`, `facebook_profile_posts`.
- **REST:** `GET https://api.captapi.com<path>?url=<profile URL>&limit=<N>`
  with header `Authorization: Bearer $CAPTAPI_API_KEY`. Use `limit=4` for
  Facebook; a higher value is a 422 validation error.

Failed calls cost 0 credits. Retry a `retryable: true` error once; record a
`retryable: false` error as the answer for that platform.

## Steps

1. **Pull listings** for every competitor × platform the user provided.
2. **Filter to the period** by each post's `publishedAt` — a listing returns
   the newest posts however old they are. Report "no posts in this period"
   separately from a failed call. For X, keep the tweets inside the period
   and note that the source is a popularity-ranked set.
3. **Merge cross-posts:** posts from one competitor on different platforms
   with timestamps a few minutes apart and overlapping text are one item.
4. **Score each post** against its own account on the same platform:
   `engagement = likes + comments (+ shares/reposts when present)`;
   `lift = engagement / median engagement of that account's pulled posts`.
   Posts with lift ≥ 2 are "outperformers".
5. **Tag each post** with format (short video, long video, image, carousel,
   text, link) and a 2–4 word topic you infer from the caption.
6. **Compare with the previous digest** if provided: posting frequency change,
   new topics, formats dropped or added.

## Digest format

1. **Activity table:** competitor × platform → posts this period (and change
   vs last period).
2. **Outperformers:** up to 5 per competitor — platform, date, format, first
   line of caption, engagement and lift, link, and source endpoint.
3. **Themes** `[needs human review]`: 3–5 topics or formats competitors are
   leaning into, each backed by at least two linked posts.
4. **Gaps and caveats:** platforms not provided or failed, the Facebook and X
   notes above, and approximate counts.
5. Credit total for the run.

## Where a human must review

- Topic tags and themes — they are inferred from captions.
- Any "strategy" reading of the data; the digest shows what was posted and
  how it performed relative to the account, not why.
