# Muapi social capability map

Use this file to choose a currently exposed capability and to keep provider
coverage separate from the broader social workflows. The host must still
verify availability and permissions at runtime.

## Account and publishing surface

| Capability | Surface | Scope | Provider/back end | Important boundary |
| --- | --- | --- | --- | --- |
| `social.list_accounts` | `GET /social/accounts` | Read connected accounts and IDs | Muapi social layer | `connected` must be true and the platform must match the intended target |
| `social.publish` | `POST /social/publish` | Publish or schedule one media item to one account | Muapi social layer; underlying platform back ends vary | Write action; requires explicit approval and a connected account |
| Scheduled-post management | `GET /social/posts`, `PATCH /social/posts/{id}`, `DELETE /social/posts/{id}` | Inspect, edit, or cancel scheduled posts | Muapi social layer | PATCH and DELETE are write actions and require re-approval when content or timing changes |

The platform-specific asynchronous publish tasks are:

They use the host's async submit/poll contract: `POST /api/v1/{task}` returns
a request ID, and the host polls `/api/v1/predictions/{request_id}/result` or
uses an approved webhook. Do not treat submission as final publication.

| Task | Platform | Provider/back end | Media/output boundary |
| --- | --- | --- | --- |
| `youtube-publish` | YouTube | Direct Google OAuth | Public video URL; returns a request ID and a completed video URL |
| `tiktok-publish` | TikTok | PostForMe-backed social layer | Public video URL; returns a request ID and publish result |
| `instagram-publish` | Instagram Business | PostForMe-backed social layer | Public image/video URL; placement and Reel options vary |
| `facebook-publish` | Facebook Page | PostForMe-backed social layer | Public image/video URL; page/account must be connected |
| `linkedin-publish` | LinkedIn | PostForMe-backed social layer | Exposed by the catalog, but connection/provider success must be verified at runtime |
| `x-publish` | X | PostForMe-backed social layer | Public image/video URL and caption |
| `threads-publish` | Threads | PostForMe-backed social layer | Public image/video URL and placement |
| `pinterest-publish` | Pinterest | PostForMe-backed social layer | Public image/video URL, board, title, and link as supported |

Additional YouTube write tasks may be exposed for `youtube-set-thumbnail` and
`youtube-update-metadata`. Treat them as separate approved writes and verify
the connected account's current OAuth scope before using them.

## Public retrieval tasks

The current Muapi catalog exposes narrower public retrieval tasks. They do not
form a universal mention, creator-search, sentiment, or owned-account
analytics API.

`social.read_posts` is live on Muapi and covers known-account retrieval for
TikTok, Instagram, Facebook, and Reddit (Reddit is subreddit-level, not
per-user — the `username` field means "subreddit name" for `platform=reddit`)
from a single capability. **LinkedIn personal-post retrieval is not
supported** (only a separate, unrelated LinkedIn company-page capability
exists upstream). Like the tasks below, `social.read_posts` requires a known
account/subreddit identifier per platform — it is not a keyword, hashtag, or
cross-platform mention search, and it does not return sentiment or theme
classification.

`social.search_posts` is live on Muapi and covers keyword/hashtag search for
**TikTok, Instagram, and YouTube only** — a leading `#` on Instagram searches
by hashtag, anything else searches Reels by keyword. **Reddit and LinkedIn
have no routed search capability at all** (confirmed against the upstream
catalog) and are rejected at the schema level rather than silently failing.
X/Threads are not covered (a Threads search route exists upstream but errors
in testing, so it is not exposed). This is content search, not a creator/user
search — it surfaces posts matching a query, not creator profiles or
follower/audience data.

| Task | Provider | Required lookup | Useful evidence |
| --- | --- | --- | --- |
| `tiktok-fetch-profile` | TOKAPI | TikTok username | profile name and follower count when returned |
| `tiktok-fetch-videos` | TOKAPI | TikTok username | recent videos, URLs, timestamps, views, likes, comments, and shares when returned |
| `instagram-fetch-reels` | INSTAGRAM_SCRAPER | Instagram username | recent Reels, URLs, timestamps, views, likes, comments, and shares when returned |
| `youtube-fetch-shorts` | YOUTUBE_SCRAPER | channel ID or search query | Shorts/search results, timestamps, views, likes, comments, and shares when returned |
| `twitter-fetch-posts` | TWITTER_SCRAPER | X username | recent posts, URLs, timestamps, views, likes, comments, and shares when returned |
| `facebook-fetch-reels` | FACEBOOK_SCRAPER | Facebook username or page slug | recent Reels, URLs, timestamps, views, likes, comments, and shares when returned |

Inputs and limits vary by task. Preserve the exact username, channel ID,
query, count, cursor, date filters, country/language, and sort options used.
Use pagination only when the host exposes the returned cursor.

## Creator search, profile, analytics, and lookalike

These four are coded on Muapi's server but **not yet confirmed live in
production** (pending a DB sync) — verify availability at runtime before
assuming any of them are callable; if one 500s as not-initialized, fall back
to `social.search_posts` lead-sourcing or a host-supplied candidate list and
say so explicitly. All four cover **Instagram, TikTok, and YouTube only** —
X and Facebook have no equivalent and never silently degrade into a
content-fetch substitute; state that plainly instead.

| Capability | Surface | Required lookup | What it returns |
| --- | --- | --- | --- |
| `social.search_creators` | `POST /creators-search` | A natural-language query and/or structured filters (niche, platform, country, min/max followers, min engagement rate) — at least one required | Matching creator profiles, merged across two providers when both a query and structured filters are given |
| `social.creator_profile` | `POST /creators-profile` | Platform + creator handle | That creator's profile, contacts, and stats |
| `social.creator_analytics` | `POST /creators-analytics` | Platform + creator handle | That creator's own performance analytics and audience demographics (third-party creator data, not the requester's owned-account insights — see the host-supplied analytics contract below) |
| `social.creator_lookalike` | `POST /creators-lookalike` | Platform + reference creator's handle | Up to 20 creators similar to the reference creator |

`social.search_creators` is a real creator/profile database search — distinct
from `social.search_posts`, which only surfaces post *content* matching a
query. Once confirmed live, treat `social.search_creators` as the actual
discovery path on Instagram/TikTok/YouTube, and `social.search_posts` as a
fallback/supplementary lead-generation aid rather than the primary discovery
tool for those three platforms.

## Not currently available as generic Muapi capabilities

Do not call or describe these as live unless the host exposes a verified
replacement:

- Creator search, profile, analytics, or lookalike coverage for **X or
  Facebook** — the four capabilities above are Instagram/TikTok/YouTube only;
- `social.sentiment_analysis` for a provider-returned sentiment or theme
  result.

`social.search_posts` and `social.search_creators` together close real
cross-account discovery gaps on TikTok, Instagram, and YouTube, but the
retrieval and search tasks together still cannot establish complete brand
mention volume, share of voice, Reddit/X/LinkedIn coverage, community
norms/comments, or a fully global trend. The host may classify supplied text
for themes or sentiment, but label that result `assistant-derived`, include
the sample and method, and do not present it as a Muapi sentiment metric.

## Host-supplied analytics contract

Muapi public post metrics are not the same as owned-account insights. If the
user asks about reach, impressions, profile visits, link clicks, conversions,
revenue, or account-level performance **for their own connected account**,
the host must supply a read-only platform analytics connector or an export
from the relevant platform. GA4 or another analytics system can add
landing-page, campaign, and conversion evidence; it cannot reconstruct
missing platform impressions. This is separate from `social.creator_analytics`
above, which returns a *third-party candidate creator's* own audience
demographics/performance data on Instagram, TikTok, or YouTube — not the
requester's owned-account insights, and not a substitute for this contract
when the question is about the requester's own account.

Useful analytics fields include:

- platform, account, post ID or URL, reporting timezone, date range, and
  attribution window;
- impressions/reach/views, likes, comments, shares, saves, watch time or
  completion, and follower change when the platform supplies them;
- profile visits, outbound clicks, campaign parameters, landing page, key
  events, conversions, and revenue when connected analytics supplies them;
- export filters, sampling, consent limitations, and whether the result is
  account-level or public-post data.

If no connector exists, accept a CSV, JSON, dashboard export, or host query
result supplied by the user. Preserve the original, record its export date,
and mark unsupported measures unavailable. Never ask for an API key in a
report or infer account analytics from public engagement counts.

## Result and cost rules

Check the host's current catalog or make a bounded test call before a broad
run. Preserve request IDs, provider names, cost, timestamps, raw responses,
and errors. A zero-result response is zero only when the requested scope was
actually supported and the response was complete; an unsupported platform,
missing account, missing username, failed provider, or partial sample is
unknown.
