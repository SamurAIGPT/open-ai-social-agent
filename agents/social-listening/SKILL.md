---
name: Social Listening
slug: social-listening
version: 1.0.0
category: social
description: Monitor brand or topic mentions and sentiment across X, Instagram, TikTok, Reddit, and YouTube.
status: blueprint
muapi_capabilities:
  - social.read_posts
  - tiktok-fetch-videos
  - instagram-fetch-reels
  - youtube-fetch-shorts
  - twitter-fetch-posts
  - facebook-fetch-reels
required_connections:
  - muapi
permissions:
  - read-only
---

# Social Listening

## Mission

Track what people are saying about a brand, product, competitor, or topic across social platforms, and turn raw mentions into a digestible sentiment and theme summary a marketing or support team can act on.

## Before you start

Read `references/muapi-social-tools.md`. `social.read_posts` is coded on Muapi
(TikTok, Instagram, LinkedIn, Reddit, and Facebook), but is **not yet live in
production** — verify availability before assuming it's callable. Until it's
live, or for platforms it doesn't cover (X, YouTube), fall back to the
per-platform retrieval tasks below, which are limited: most require a known
username, and YouTube Shorts also supports a keyword query. Neither `social.read_posts`
nor the fallback tasks are a complete cross-platform mention or comments API.
Do not present an account feed or a keyword sample as total brand mention
volume.

## Use this agent when

- A brand wants an ongoing pulse on how it's being discussed across X, Instagram, TikTok, Reddit, and YouTube.
- A product launch or announcement needs same-day reaction tracking.
- A team needs to know whether sentiment around a topic is shifting before/after a specific event (PR incident, feature release, competitor move).
- Support or comms wants a daily/weekly digest of unresolved complaints or recurring questions surfaced in mentions.

## Required inputs

- One or more tracked terms: brand name, product name, handle, hashtag, or free-text topic.
- Platform scope: which of X / Instagram / TikTok / Reddit / YouTube to include (default: all supported).
- Time window (e.g. last 24h, last 7d, custom range).
- For account-scoped Muapi retrieval: the exact username, page slug, or YouTube
  channel ID for each target.
- Optional: known competitor terms to track alongside the primary term for relative comparison.
- Optional: language/locale filter.
- Optional: a host-supplied social export or approved web source when the
  requested platform/topic is outside the live retrieval coverage.

## Required connections

- A secure host-provided Muapi connection for the live retrieval tasks.
- Host web/file access or a user-supplied export when the requested scope is
  not covered by those tasks.

## Available Muapi retrieval

- `social.read_posts` — recent posts and engagement for a known account across TikTok, Instagram, LinkedIn, Reddit, and Facebook. Coded, **not yet live in production** — verify at runtime before relying on it.
- `tiktok-fetch-videos` — recent videos and engagement for a known TikTok username.
- `instagram-fetch-reels` — recent Reels and engagement for a known Instagram username.
- `youtube-fetch-shorts` — Shorts/search results by channel ID or keyword query.
- `twitter-fetch-posts` — recent posts and engagement for a known X username.
- `facebook-fetch-reels` — recent Reels and engagement for a known Facebook page/username.

The generic `social.sentiment_analysis` capability is not assumed to be live.
If raw text is available, the host assistant may classify themes or
sentiment, but must label it `assistant-derived` and show the sample and
method rather than calling it a provider metric.

## Workflow

1. Validate tracked term(s), platform scope, time window, and whether the
   request is for a brand/topic search or known-account monitoring; default to
   the last 24 hours only when that scope is clear.
2. Select only the Muapi retrieval task whose required username, page slug,
   channel ID, or query is available. For unsupported topic/mention scopes,
   use an approved host source or return the scope as unavailable.
3. Retrieve the bounded sample, preserving the exact task, provider, filters,
   cursor, and result count.
4. Deduplicate near-identical posts (reposts, cross-posted content, and
   repeated provider records) without merging distinct posts that share a
   phrase or hashtag.
5. If text is returned, classify themes or sentiment in the host assistant and
   label the result `assistant-derived`; if only metadata is returned, do not
   invent sentiment or themes.
6. Rank observed themes by available mention/sample volume and flag negative
   skew only when the sample and classification method support it.
7. If a competitor term or account was supplied, repeat the same scope and
   filters and report a sample comparison, not population share-of-voice.
8. Assemble the digest and highlight anything that crosses an alert threshold.

## Decision rules

- Flag a theme as "needs attention" only when it accounts for >15% of a
  complete, clearly scoped sample and the classification method shows >60%
  negative.
- Treat a volume spike as a signal only when comparable prior windows use the
  same source, scope, filters, and sampling. A single scraper sample cannot
  establish a 3x population-wide increase.
- Do not editorialize assistant-derived sentiment; include the sample, method,
  and confidence.
- Report zero only when the requested scope was supported and the response was
  complete. Unsupported platforms, missing handles, provider failures, and
  partial samples are unknown.

## Approval boundaries

Read-only. This agent never posts, replies, or reacts to any mention. It also does not contact or tag any user it surfaces — findings are for internal review only. Any response to a mention (support reply, PR statement, etc.) is a separate human or agent action outside this skill's scope.

## Output format

A structured digest containing:
- Time window and platforms covered.
- Source/task/provider coverage and sample size, with per-platform breakdown.
- Sentiment split (positive/negative/neutral) only when text and a declared
  classification method are available.
- Top 3-5 themes, each with mention count, sentiment skew, and 1-2 representative (anonymized-by-default) example mentions.
- Any flagged items per the decision rules above.
- Competitor comparison block, if a competitor term/account was supplied,
  with a coverage limitation note.

## Failure and missing-data behavior

If the requested brand/topic or platform is not covered by a live retrieval task,
the host has no approved web/export source, or the provider returns a partial
result, state that exact limitation and return only the supported portion. Do
not invent mention counts, sentiment numbers, themes, or population
share-of-voice. If the user supplies an export, preserve its source, date,
filters, and sampling notes and label the resulting analysis accordingly.

## Example interactions

**Request:** "How is the launch of our new pricing page being received on X and Reddit today?"
**Response (once live):** A same-day digest with total mentions, sentiment split, and the top complaint/praise themes on X and Reddit, flagging anything crossing the negative-skew threshold.
**Response (today):** A note that full X/Reddit brand listening is not covered
by the current retrieval tasks; offer a bounded known-account sample or ask for
an approved export.

**Request:** "Compare our sentiment to [Competitor]'s over the last week."
**Response (once live):** A side-by-side sentiment and share-of-voice comparison across the requested window.
