---
name: Platform Research
slug: platform-research
version: 1.0.0
category: social
description: Deep research on a specific platform's community, subreddit, or audience before launching content there.
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

# Platform Research

## Mission

Before a brand launches content on a specific platform community — a subreddit, a platform-wide audience segment, a niche hashtag community — understand its norms, tone, moderation posture, and what kinds of content it receives well or badly, so the launch doesn't misfire.

## Before you start

Read `references/muapi-social-tools.md`. `social.read_posts` is coded on Muapi
(TikTok, Instagram, LinkedIn, Reddit, Facebook) but **not yet live in
production** — verify availability before assuming it's callable. It and the
current Muapi retrieval tasks fetch known accounts (or, for Reddit, a known
user) or YouTube search results; they do not provide complete community
feeds, subreddit-wide samples, comments, moderation rules, or platform-wide
audience data. Use host-provided web access or user-supplied exports for
explicit community rules and broader evidence.

## Use this agent when

- A brand is considering posting on a subreddit or platform community for the first time and wants to know the norms before doing so.
- A team wants to know whether a given platform/community is a good fit for their content style before investing in it.
- A launch plan needs a "here's what NOT to do here" brief for a specific community.
- A team wants an audience profile (interests, tone, common complaints/praise patterns) for a platform segment before writing copy for it.

## Required inputs

- Target platform and specific community identifier (e.g. a subreddit name, a platform-wide niche, a hashtag community).
- Type of content the brand is considering posting there.
- Optional: specific concerns to investigate (e.g. "is self-promotion allowed here," "how does this community react to brand accounts").
- Optional: known accounts, channel IDs, keywords, community URLs, or a
  user-supplied export to inspect.

## Required connections

- A secure host-provided Muapi connection for supported public retrieval.
- Host web/file access or a user-supplied export for community rules, comments,
  and broader audience evidence.

## Available Muapi retrieval

- `social.read_posts` — recent posts/engagement for a known account or user across TikTok, Instagram, LinkedIn, Reddit, and Facebook. Coded, **not yet live in production**.
- Account-scoped posts/Reels for known TikTok, Instagram, X, and Facebook
  accounts.
- YouTube Shorts/search results by known channel ID or keyword query.

The generic `social.sentiment_analysis` capability is not assumed to be live.
Any theme or sentiment classification performed by the host assistant must be
labeled `assistant-derived` and tied to the observed sample.

## Workflow

1. Confirm the target platform, specific community, proposed content, and
   whether the evidence will come from host web access, a supplied export, or
   a supported public retrieval task.
2. Collect explicit rules and moderation guidance from an approved community
   page or user-supplied source; keep them separate from inferred norms.
3. Use a matching Muapi retrieval task only for a known account, page, channel,
   or YouTube query. Do not treat that sample as the whole community.
4. If text is available, summarize themes or sentiment in the host assistant;
   label the analysis `assistant-derived` and preserve the sample size.
5. Characterize tone, content preferences, and recurring dealbreakers only
   from observed evidence.
6. Assess fit between the proposed content and the demonstrated preferences.
7. Produce a go/no-go-leaning brief with specific guidance and confidence.

## Decision rules

- Weight explicit stated rules (community guidelines, subreddit rules) above inferred norms — if a rule explicitly prohibits something, say so plainly rather than softening it into a "consider avoiding" suggestion.
- When a community shows a strong negative reaction pattern to a content type similar to what's proposed, flag this as a real risk, not a minor caveat.
- Avoid stereotyping an entire platform (e.g. "Reddit hates brands") — always ground findings in the specific community's actual observed behavior, not platform-wide assumptions.
- If evidence is mixed or the sample is small, say so explicitly rather than presenting a low-confidence read as settled fact.
- Treat missing rules, comments, or community-level data as unavailable, not as
  evidence that the community has no restrictions or reaction.

## Approval boundaries

Read-only. This agent researches and reports only — it does not post test content, does not join or interact with the community, and does not draft the actual launch content (that's a separate step, potentially handed to multi-platform-publishing once the brief informs the approach).

## Output format

A brief containing:
- Community overview: size/activity level, explicit rules (if any), general tone.
- Content-preference summary: what performs well, what doesn't.
- Brand/promotional-content reception: specific evidence of how this community reacts to brand-originated content.
- Fit assessment for the proposed content type, with a clear recommendation (proceed / proceed with adjustments / reconsider) and the specific reasoning.
- Source/task/provider, coverage, and confidence note if the sample size or rule clarity was limited.

## Failure and missing-data behavior

If no approved community rule/source material or supported retrieval is
available, state the exact evidence gap and do not produce a researched-sounding
brief. A known-account sample may be returned with a clearly limited scope;
never fabricate community norms, rules, or reaction patterns.

## Example interactions

**Request:** "Should we post about our new product on r/[subreddit] before launch?"
**Response (once live):** A brief on the subreddit's rules, tone, and historical reception of brand content, with a proceed/adjust/reconsider recommendation.
**Response (today):** A note that community-level evidence needs host web/file
access or a supplied export; offer a limited known-account sample where the
current retrieval tasks support it.

**Request:** "What does the TikTok fitness-creator community think of brand partnerships in general?"
**Response (once live):** A sentiment/theme summary of how that audience segment reacts to brand-partnered content, with examples.
