---
name: creator-vetting-transcripts
description: Vet a creator before a sponsorship or partnership by pulling their public TikTok, Instagram and YouTube profiles, recent posts, and transcripts of what they actually said, then summarising audience size, posting rhythm, engagement, past sponsors and any brand-safety flags. Uses the Captapi API (MCP tools or REST with CAPTAPI_API_KEY). Shows a credit estimate first and cites the endpoint behind every claim. Use when someone asks to vet, audit, shortlist or background-check an influencer or creator.
---

# Creator vetting with transcripts

Build a one-page brief on a creator from public data: who they are on each
platform, how often they post, how their recent posts perform, which brands
they already work with, and what they say on camera.

## What this can and cannot tell you

- Only public, logged-out data. Private accounts, audience demographics,
  follower authenticity scores and DMs are not available — say so; never
  invent them.
- Engagement rate here is `(likes + comments) / followers` over the posts
  pulled. It is a rough signal from a small sample, not an audited metric.
- Some counts are rounded by the platform (e.g. "1.2M"); keep the
  `...IsApproximate` flags from the response and show "≈" for them.
- Transcripts come from speech in the video (TikTok, Instagram) or published
  captions (YouTube). A missing transcript means no speech or no captions,
  not that the creator said nothing worth checking.

## Inputs to collect first

1. The creator's profile URLs or handles on TikTok, Instagram and/or YouTube.
   Do not guess handles across platforms — ask, or mark the platform as
   "not provided".
2. How many recent posts per platform (default 10) and how many transcripts
   (default 3 most recent videos per platform).
3. What the brand cares about: competitors to look for, topics to avoid.

## Credit estimate — show before running

Per creator, with defaults (10 posts, 3 transcripts per platform):

| Platform | Calls | Credits |
|---|---|---|
| TikTok | `/v1/tiktok/channel-details` 1 · `/v1/tiktok/channel-posts` 2 · 3 × `/v1/tiktok/transcript` 2 | 9 |
| Instagram | `/v1/instagram/channel-details` 1 · `/v1/instagram/channel-posts` (0.3 per post, 10 posts = 3) · 3 × `/v1/instagram/transcript` 2 | 10 |
| YouTube | `/v1/youtube/channel-details` 1 · `/v1/youtube/channel-videos` 2 · 3 × `/v1/youtube/transcript` 1 · `/v1/youtube/video-sponsors` 0 | 6 |

All three platforms: about **25 credits** per creator; up to ~35 with more
posts or transcripts. Print the estimate for the requested creators, check
`GET /v1/account/balance` (free), and ask the user to confirm.

## How to call Captapi

- **MCP:** `tiktok_channel_details`, `tiktok_channel_posts`,
  `tiktok_transcript`, `instagram_channel_details`, `instagram_channel_posts`,
  `instagram_transcript`, `youtube_channel_details`, `youtube_channel_videos`,
  `youtube_transcript`, `youtube_video_sponsors`.
- **REST:** `GET https://api.captapi.com<path>?url=<profile or video URL>` with
  header `Authorization: Bearer $CAPTAPI_API_KEY`.

Failed calls cost 0 credits. A 403 such as `profile_restricted` or a 404 is a
final answer for that platform — record it and move on.

## Steps

For each platform the user provided:

1. **Profile** — `channel-details` with the profile URL. Keep followers,
   following, post count, bio, bio links, verified flag, account type.
2. **Recent posts** — `tiktok/channel-posts`, `instagram/channel-posts` or
   `youtube/channel-videos` with `limit=<N>`. From the rows compute: posts per
   week over the window, median views (when present), median likes and
   comments, engagement rate, and the best and worst post.
3. **Sponsor signals** from the same rows:
   - Instagram rows carry `isPaidPartnership`; list those posts and the brands
     they mention.
   - TikTok and YouTube: scan captions/titles for `#ad`, `#sponsored`,
     "paid partnership", discount codes and brand @mentions.
   - YouTube: `youtube/video-sponsors?url=<video URL>` for each transcribed
     video returns community-submitted sponsor segments (free). Label them as
     community-reported.
4. **Transcripts** — for the most recent videos, `tiktok/transcript`,
   `instagram/transcript` or `youtube/transcript` with the video URL. Read
   them for: products and brands named, competitor mentions, claims that would
   need substantiation (health, finance), and language or topics the brand
   listed as off-limits.

## Brief format

1. **Snapshot table:** one row per platform — handle, followers, posts/week,
   median views, engagement rate, verified. Each cell traceable to its
   endpoint.
2. **Past sponsors:** brand, platform, post date, link, and how it was
   detected (`isPaidPartnership`, caption tag, community segment).
3. **What they say:** 3–5 bullets drawn from transcripts, each with a short
   quote and the video link.
4. **Flags** `[needs human review]`: competitor work, risky claims,
   off-limits topics, sudden follower jumps between profile and post data.
5. **Gaps:** platforms not provided, restricted, or with no transcripts.
6. Credit total for the run.

## Where a human must review

- Every flag — transcripts can be misheard and context matters.
- Whether a detected "sponsor" was paid or just a mention.
- Fit with the brand: the brief gives evidence, not a hire / no-hire verdict.
