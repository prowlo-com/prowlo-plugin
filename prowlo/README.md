# Prowlo for Claude

Connect Claude to a curated, scored Reddit opportunity feed. Browse high-intent threads, read subreddit moderation patterns, manage tracked keywords, and draft replies — all from inside Claude.

Prowlo is a Reddit lead intelligence platform. This plugin gives Claude direct access to your Prowlo workspace through the Model Context Protocol (MCP), plus a skill that teaches Claude how to engage on Reddit safely, plus two slash commands that drive the full opportunity-to-draft workflow from a single prompt.

## What you get

- **MCP connector** — `prowlo` server hosted at `https://api.prowlo.com/mcp`. 10 tools covering opportunity intelligence and keyword management.
- **`reddit-engagement` skill** — loads automatically when you ask Claude anything about Reddit replies, subreddit safety, or lead drafting. Encodes Prowlo's dual-scoring framework, the survival-rate-over-strictness-label rule, and the "draft, then human-review" workflow.
- **`/prowlo-feed`** — show today's top-scored opportunities, filterable by subreddit, search text, opportunity threshold (HIGH/MEDIUM/LOW), risk threshold (HIGH/MEDIUM/LOW), relevance preset, or limit.
- **`/prowlo-engage <postId>`** — run the full qualify → read thread → draft → submit workflow on a single opportunity. `postId` is a UUID returned by `/prowlo-feed`.

## Install

1. Click **Install** in the Claude plugins marketplace, or drop the `.plugin` file into Cowork.
2. Authorize Prowlo when Claude prompts you. Cowork uses OAuth2 — no API keys, no JSON config. Claude Desktop and Claude Code use a Bearer token; generate one at <https://app.prowlo.com/settings/api-keys>.
3. Try a first prompt: *"Show me today's top Reddit opportunities."*

## Requirements

- A Prowlo account. New here? Start a free 7-day trial at <https://prowlo.com/register>.
- A Claude product that speaks the Model Context Protocol — Cowork, Desktop, Code, Cursor, or Windsurf.

## Tools exposed by the MCP

**Opportunity intelligence**
- `get_opportunities` — list scored opportunities with filters: `opportunityThreshold` (HIGH/MEDIUM/LOW), `riskThreshold` (HIGH/MEDIUM/LOW), `relevancePreset` (high/medium/all), `keywords` (full-text search), `limit`, `page`. Returns enum-level scores, intent direction, freshness, saturation, replyType, and per-post engagement guidance.
- `get_opportunity` — full detail for one opportunity by `postId` (UUID). Includes post body, all signals, full guidance, and any pre-drafted reply (`autoDraft`).
- `read_comments` — comments under a post by `postId`, for thread context before drafting.
- `get_subreddit_intelligence` — moderation patterns, rules with enforcement levels, survival rates (self-mention, links), trigger words with risk scores, account readiness thresholds. Pass a subreddit name for detail; omit it for an overview of all profiled subreddits.
- `get_product_profile` — your positioning, target audience, voice, and engagement guidelines.
- `submit_draft` — send a reply draft to your Prowlo dashboard for human review.

**Keyword management**
- `keyword_list` — currently tracked keywords with match counts.
- `keyword_create` — add a tracked keyword (Starter: 10 slots, Growth: 30). Note: changes take effect on the next scheduled processing cycle.
- `keyword_update` — activate or pause a keyword (preserves history).
- `keyword_delete` — remove a keyword and its match history.

## What you can ask

These prompts work against the real API shapes — enum thresholds, UUID `postId`s, and the rich signals fields the MCP returns:

- *"Show me today's high-intent Reddit threads — opportunity HIGH, risk no higher than MEDIUM."*
- *"List my latest BUYING-intent opportunities in r/entrepreneur and rank them by replyType."*
- *"Pull the full context for postId `bf266d7c-a876-4ec9-b123-b65be7220cb3` and draft a reply that fits the subreddit."*
- *"Which subreddits in my feed have self-mention survival under 20%? I'll skip those this week."*
- *"Add 'headless CMS' as a tracked keyword. Pause 'CMS' — too noisy."*
- *"Summarize the moderation pattern in r/startups: survival rate, trigger words, account readiness."*

## Safety model

- **No auto-posting.** `submit_draft` writes to your Prowlo dashboard. You review and post manually from your Reddit account. The plugin cannot publish to Reddit.
- **No Reddit credentials in Claude.** Claude never sees your Reddit password, cookies, or tokens. The MCP exposes data Prowlo has already ingested via its read-only Reddit pipeline.
- **Survival rates beat labels.** The `reddit-engagement` skill teaches Claude to read `survivalRates.selfMention`, not `strictnessLevel`. A "lenient" subreddit can still remove 93% of self-promotional replies — the skill catches that.
- **Revocable.** OAuth access can be revoked from your Prowlo dashboard. API keys can be rotated at any time.

## Pricing

The MCP is included on every paid Prowlo plan at no extra cost, including the 7-day free trial. Plan limits apply (opportunity volume, tracked keywords, seats).

## Support

- Setup guide: <https://prowlo.com/help/mcp-setup>
- Integration page: <https://prowlo.com/integrations/claude-mcp>
- Help center: <https://prowlo.com/help>
- Email: support@prowlo.com

## License

Proprietary. © Prowlo.
