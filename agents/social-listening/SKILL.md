---
name: Social Listening
slug: social-listening
version: 1.0.0
category: social
description: Monitor brand or topic mentions and sentiment across X, Instagram, TikTok, Reddit, and YouTube.
status: blueprint
muapi_capabilities:
  - social.search_posts
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

Read `references/muapi-social-tools.md`. `social.search_posts` is live on
Muapi and is the primary tool for this agent — keyword/hashtag search across
TikTok, Instagram, and YouTube (Instagram: a leading `#` searches by
hashtag). It does not cover X, Reddit, LinkedIn, or Threads. `social.read_posts`
is also live (TikTok, Instagram, Facebook, and Reddit as subreddit-level;
LinkedIn personal posts unsupported) but is account-scoped, not a mention
search — use it to pull a known account/subreddit's own posts, not to find
who is talking about a topic. For platforms neither tool covers (X, and
comments/community discussion on any platform), fall back to the
per-platform retrieval tasks below or an approved host source. None of these
tools together are a complete cross-platform mention or comments API. Do not
present a keyword-search sample or an account feed as total brand mention
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

- `social.search_posts` — keyword/hashtag search for public posts on TikTok, Instagram, and YouTube. Live. This is the tool for topic/brand-term tracking; it does not cover X, Reddit, LinkedIn, or Threads, and it returns matching posts, not comments or sentiment.
- `social.read_posts` — recent posts and engagement for a known account across TikTok, Instagram, Facebook, and Reddit (subreddit-level). Live; LinkedIn personal posts unsupported.
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
2. For topic/brand-term tracking on TikTok, Instagram, or YouTube, call
   `social.search_posts` with the tracked term (or `#term` for an Instagram
   hashtag). For known-account monitoring, select the retrieval task whose
   required username, subreddit, page slug, or channel ID is available. For
   X, Reddit-wide, LinkedIn, or comment-level scopes, use an approved host
   source or return the scope as unavailable.
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
**Response:** A note that X and Reddit-wide brand listening are not covered
by `social.search_posts` (TikTok/Instagram/YouTube only) or `social.read_posts`
(known-account/subreddit only); offer a `social.search_posts` sample on
TikTok/Instagram/YouTube instead, a bounded known-account/subreddit sample, or
ask for an approved export.

**Request:** "What's being said about our new product on TikTok and Instagram this week?"
**Response:** A same-day digest built from `social.search_posts` on both
platforms — mention sample, top themes with representative posts, and
assistant-derived sentiment labeled as such (not a provider metric).

**Request:** "Compare our sentiment to [Competitor]'s over the last week."
**Response:** A side-by-side sample comparison on TikTok/Instagram/YouTube
(same term/window/method for both), explicitly labeled a sample comparison,
not a population-wide share-of-voice measurement.
