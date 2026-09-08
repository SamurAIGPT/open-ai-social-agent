# AI Social Agent

An AI agent for social media management — listening, creator discovery, multi-platform publishing, and trend research across X, Instagram, TikTok, Reddit, YouTube, Facebook, LinkedIn, Threads, and Pinterest — backed by a secure host-provided Muapi connection and clearly scoped social data.

Part of [Agency Agents OS](https://github.com/Anil-matcha/agency-agents-os), an open ecosystem of specialized AI agents for real business work.

## Related Projects

- [Agency Agents OS](https://github.com/Anil-matcha/agency-agents-os) — the central catalog this repo is part of.
- [ai-youtube-agent](https://github.com/SamurAIGPT/ai-youtube-agent) — uses this repo's `social.publish` capability to actually upload optimized videos to YouTube.
- [ai-content-repurposing-agent](https://github.com/SamurAIGPT/ai-content-repurposing-agent) — feeds this repo's multi-platform-publishing sub-agent with clipped video.
- [ai-image-agent](https://github.com/SamurAIGPT/ai-image-agent) — supplies image posts for this repo to publish.
- [ai-marketing-agent](https://github.com/SamurAIGPT/ai-marketing-agent) — plans the campaigns this repo publishes and monitors.
- [ai-reputation-agent](https://github.com/SamurAIGPT/ai-reputation-agent) — shares this repo's sentiment/listening data sources.
- [MuAPI MCP docs](https://muapi.ai/docs/mcp) — connect this repo's `SKILL.md` files via MCP.
- [MuAPI API reference](https://muapi.ai/docs/api-reference) — request/poll pattern used by the live `social.publish` endpoint.
- MuAPI access — configure the connection and credentials through the host assistant's secure settings.

## What this covers

This repo is the umbrella for anything an agency or in-house team would call "the AI social agent": knowing what's being said about a brand or topic, finding the right creators to work with, adapting one piece of content for each platform's format, and understanding what's currently working in a niche — without manually monitoring five different apps.

## Sub-agents

| Agent | Does | Status |
|---|---|---|
| [Social Project Setup](agents/social-project-setup/SKILL.md) | Establish brand, account, campaign, approval, and measurement context | Ready |
| [Social Listening](agents/social-listening/SKILL.md) | Monitor brand/topic mentions and sentiment across platforms | Blueprint — scoped retrieval only |
| [Creator Discovery](agents/creator-discovery/SKILL.md) | Find relevant creators/influencers for a campaign by niche and audience fit | Blueprint — candidate validation only |
| [Multi-Platform Publishing](agents/multi-platform-publishing/SKILL.md) | Adapt and schedule one media post across YouTube, TikTok, Instagram, Facebook, LinkedIn, X, Threads, and Pinterest | Blueprint |
| [Trend Discovery](agents/trend-discovery/SKILL.md) | Surface what's currently working/trending in a niche to inform content strategy | Blueprint — limited platform/query coverage |
| [Platform Research](agents/platform-research/SKILL.md) | Deep research on a platform's community/subreddit/audience before launching content there | Blueprint — host web/data required |

## Muapi capability status

Live account and publishing surfaces include:

- `social.list_accounts` (`GET /social/accounts`) — list connected social accounts and their connection state.
- `social.publish` (`POST /social/publish`) — publish or schedule one media item to one connected account.
- Scheduled-post management: `GET /social/posts`, `PATCH /social/posts/{id}`, `DELETE /social/posts/{id}`.
- Platform-specific publish tasks for YouTube, TikTok, Instagram, Facebook, LinkedIn, X, Threads, and Pinterest, subject to runtime account/provider availability.
- Narrow public retrieval tasks for TikTok profiles/videos, Instagram Reels, YouTube Shorts, X posts, and Facebook Reels.
- `social.read_posts` — known-account post retrieval across TikTok, Instagram, LinkedIn, Reddit, and Facebook. Coded server-side on Muapi, **not yet deployed to production** — do not call it until the host confirms it's live.

The current and coded retrieval tasks do not provide complete cross-platform
brand listening, creator discovery, community feeds, or provider-returned
sentiment. The generic `social.search_creators` and `social.sentiment_analysis`
names must not be called unless the host exposes a verified implementation.
See [the capability map](references/muapi-social-tools.md) for task/provider
coverage and safe fallbacks.

Owned-account reach, impressions, audience demographics, conversions, and
revenue require platform-native analytics or GA4/other analytics supplied by
the host. Public post metrics are not a substitute.

## Setup

1. Configure the Muapi connection in the host assistant's secure settings; never paste an API key into a prompt, repository, or `.social/` artifact.
2. Connect the required platform accounts through the host/Muapi OAuth flow when publishing is needed.
3. Read `AGENTS.md`, then load the `SKILL.md` for the sub-agent you need into your agent runtime (hosted agent, MCP client, or custom LLM app), or follow it manually.


## Using with an AI agent

Every sub-agent's `SKILL.md` is model- and runtime-agnostic — it's plain Markdown, so it works with any LLM agent, not just Claude. Two integration paths:

**As an MCP connection (the agent gets live Muapi tools):**

Muapi runs an MCP server that the host assistant can connect to through its
secure connector settings. Use the host's supported header or
environment-variable mechanism; do not embed keys in connector URLs, skill
files, prompts, logs, or project artifacts.

The host should expose only the capabilities and account scope the requester
has authorized.

**As agent instructions (any LLM follows the workflow directly):**

Drop a sub-agent's `SKILL.md` into a Claude Code project's `.claude/skills/` directory, paste it into a custom-GPT/Project's system instructions, hand it to an autonomous agent framework as a tool spec, or attach it directly in a chat conversation — then ask the agent to follow it.

## Read-only vs. write actions

Social listening, creator discovery, trend discovery, platform research, and
analytics review are `read-only` — they surface information, they don't post
anything. Multi-platform publishing is `requires-approval-to-publish` — a
draft is prepared for each platform, but nothing is published or scheduled
until a human explicitly approves it.

## Status and limitations

Multi-Platform Publishing is **Blueprint** — built on `social.list_accounts` and
`social.publish`, with platform-specific publish tasks available subject to
runtime account/provider status. It publishes to eight target platforms, not
Reddit.

The four research sub-agents (Social Listening, Creator Discovery, Trend
Discovery, Platform Research) are also **Blueprint** as of 2026-09-09: their
workflows are defined against Muapi's `social.read_posts` capability, which
is coded server-side but **not yet deployed to production** (no live vendor
token, no DB sync yet). `social.read_posts` is a known-account/user post
retrieval capability, not a search, discovery, or sentiment API — it does not
by itself resolve these sub-agents' stated gaps around cross-platform mention
search, unknown-creator discovery, subreddit-wide community sampling, or
provider-returned trend/sentiment data. Each sub-agent's own `SKILL.md` states
this limitation plainly. Status will move to Tested once `social.read_posts`
is live and a real end-to-end run has been verified.

## Contributing

See [Agency Agents OS CONTRIBUTING.md](https://github.com/Anil-matcha/agency-agents-os/blob/main/CONTRIBUTING.md).

## License

[MIT](LICENSE)
