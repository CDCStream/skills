# Captapi skills

Agent skills for social-media and ad-library research with the
[Captapi API](https://captapi.com). Each skill is a `SKILL.md` an agent loads
when the task matches: it says which endpoints to call, what they cost, what
the data can and cannot show, and where a human has to check the result.

| Skill | What it does | Typical credits |
|---|---|---|
| [`ad-library-weekly-diff`](skills/ad-library-weekly-diff/SKILL.md) | Competitor ads across the Meta, Google, LinkedIn and TikTok ad libraries; new vs stopped since last week | ~11 per competitor per country |
| [`creator-vetting-transcripts`](skills/creator-vetting-transcripts/SKILL.md) | TikTok / Instagram / YouTube profile, recent posts, transcripts and past sponsors for one creator | ~25 per creator |
| [`competitor-content-digest`](skills/competitor-content-digest/SKILL.md) | Latest posts from competitors on seven platforms, ranked against each account's own baseline | ~14 per competitor |

Every skill prints a credit estimate and asks before it spends, cites the
endpoint behind each claim, and marks interpretations as needing human review.

## Install

```bash
npx skills add captapi/skills
```

List them first, or install one:

```bash
npx skills add captapi/skills --list
npx skills add captapi/skills --skill creator-vetting-transcripts
```

## Connect Captapi

The skills work with either:

- **MCP** — add the hosted server `https://api.captapi.com/mcp` to your agent
  with the header `Authorization: Bearer <your API key>`; the skills name the
  tools to call.
- **REST** — set `CAPTAPI_API_KEY`; the skills call
  `https://api.captapi.com/v1/...` with `Authorization: Bearer $CAPTAPI_API_KEY`.

Get a key at [captapi.com](https://captapi.com). Failed calls are not billed.

## Data scope

Public, logged-out data only. No private accounts, no audience demographics,
and no ad spend or performance numbers beyond what each public library
discloses. The skills say this in their reports instead of guessing.

## License

MIT
