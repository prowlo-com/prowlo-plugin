---
name: brand-and-competitor-monitoring
description: Use when the user wants to know what people say about their brand, product or competitors on Reddit, Hacker News or other sources Prowlo covers. For example, "is anyone talking about us", "how do people feel about Notion vs Obsidian", "find complaints about our competitor", or "set up tracking for our brand name".
---

# Brand and competitor monitoring

Find mentions of a brand and its competitors, summarize what people think, and, only when the user asks, set up ongoing tracking. Follow the `social-research` skill for tool choice and costs.

## Steps

1. **Collect the names.** Get the brand and any competitors. Include common variants (product name, domain, short forms). If the user wants to use their saved list, `get_product_profile` holds the product name and competitors. Ask about names that are also common words (for example "Notion" or "Linear"), since they bring noise.
2. **Search the Dataset first.** For each name, call `search_dataset` with `mode` "keyword", the name as `query`, and `any` set to the name and its variants. Add `from` for the period. Use `exclude` to drop known false matches. Results hide matches judged coincidental by default, which is what you want.
3. **Check Keyword Monitors.** `keyword_monitor_list` shows which names already have a daily Reddit-wide search feeding the Dataset. If a name has none, the Dataset may miss many mentions. Say so.
4. **Fill gaps with a few live reads.** If coverage is thin, run `social_search` with `platform` "reddit" (and "hackernews" if relevant), the name as `query`, `sort` "new" or "relevance", and `time` matching the period. Keep it to one or two searches per name.
5. **Read before judging.** Open the posts that carry the most weight with `get_record` or `social_get_post`. Do not judge sentiment from titles alone.
6. **Write the summary.**

## Output format

- **Coverage:** sources and period, number of mentions found per name, and whether a Keyword Monitor exists for each.
- **Sentiment per name:** rough split of positive, negative, mixed or neutral mentions, with the counts. Make clear this is your reading of the posts.
- **What people praise and what they complain about:** grouped themes, each linked to example posts.
- **Comparisons:** threads where people compare the brand with competitors, with links and the reasons people give.
- **Open questions:** things people asked about the brand that went unanswered, with links.

## Setting up tracking (only when asked)

If the user asks to track mentions going forward, propose the exact setup and wait for a yes before each write:

- **Daily Reddit-wide search:** `keyword_monitor_create` with `query` (the name), a short `description` that says what the name refers to (this helps weed out coincidental matches), and optional `tags`. It uses the plan's keyword limit, so check `keyword_monitor_list` first.
- **Other sources:** `watcher_create` with `sources`, for example `{ "channel": "<subreddit>", "platform": "reddit" }`.
- **Notifications:** `alert_create` with `name`, `query.any` set to the brand and its variants (required), and `destination` (`slack: true` or `webhookIds`). Call `get_account` first to confirm Slack is connected or to get webhook ids. Add `threshold` (`count`, `window`) only if the user wants spike alerts instead of one per match.
- **Mentions over time:** `trend_create` with `name`, `query.any`, `interval` "day" and a `window` such as "30d".

Do not create several of these in one go without the user agreeing to each.

## Boundaries

This skill is for listening. Prowlo cannot post or reply, and you should not draft replies to mentions, write posts meant to look like they come from ordinary users, or suggest ways to push the brand in these threads. If the user wants to respond to something, point them to the thread link so they can decide for themselves.
