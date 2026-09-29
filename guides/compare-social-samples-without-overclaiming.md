# Compare Social Samples Without Overclaiming

This walkthrough compares observed public-post samples for a brand and a competitor. It follows the [Social Listening](../agents/social-listening/SKILL.md) and [Trend Discovery](../agents/trend-discovery/SKILL.md) skills.

## Example request

> Compare what people are posting about our launch and a competitor's launch this week on TikTok and Instagram. Summarize recurring themes and possible negative reactions.

## Check coverage first

`social.search_posts` is live for keyword/hashtag search on TikTok, Instagram, and YouTube. It returns matching posts, not comments or a complete population count. `social.read_posts` is for known accounts, not topic-wide mention discovery. X, Reddit-wide search, LinkedIn, Threads, and comment-level coverage are not covered by those tools. If the host cannot retrieve the requested scope, mark it unavailable rather than implying a cross-platform result.

## Workflow

1. Confirm the exact search terms, platforms, date window, geography/language if supported, and comparison goal. Keep brand and competitor queries parallel in scope.
2. Verify the live capability and its current request schema. Run bounded searches for each term/platform pair. Preserve the request, filters, returned count, timestamps, and exact errors when available.
3. Deduplicate reposts and cross-posts carefully. Keep the result set as a sample; do not call its size total mention volume or share of voice.
4. If post text is available, group recurring themes and classify sentiment as **assistant-derived**, including the method and sample size. If only metadata is returned, do not infer sentiment.
5. Compare like with like: same platform, query type, window, and filters. Report different coverage or partial results next to the comparison.
6. Label an emerging pattern only when multiple independent posts or creators support it. A single high-engagement post is an outlier/example, not a trend. Claim velocity only from comparable historical snapshots.
7. Return the themes, representative post links, sample counts, coverage limits, and next questions. Do not publish or reply from this workflow.

## Report template

- **Scope and as-of date:** platforms, terms, time window, filters.
- **Coverage:** source/tool per platform, returned sample size, missing scope.
- **Observed themes:** evidence links, recurrence, and sentiment method if text supports it.
- **Comparison:** like-for-like observations only; no population-level share claim.
- **Caveats:** partial results, ambiguity, and unsupported requests.

## Safety and privacy

Keep credentials in the host's secure settings, not chat or `.social/`. Anonymize example usernames in narrative summaries by default. Publishing, scheduling, editing, deleting, and contacting users remain separate write actions that need explicit human approval under [the repo instructions](../AGENTS.md).
