---
name: social-research
description: Use when the user wants to research what people say on Reddit, Hacker News, X, Mastodon or RSS feeds with Prowlo. For example, finding posts about a topic, checking what a community thinks of a product or problem, looking up a specific thread or user, or asking what their Prowlo Watchers have collected.
---

# Social research with Prowlo

Prowlo is a hosted MCP server. It gives Claude three kinds of social data. Pick tools by which one answers the question, and use the cheapest one that does.

## The three data planes

1. **Dataset (free, rate-limited).** The user's own indexed posts, collected over time by their Watchers. Watchers follow subreddits, Reddit users, Reddit-wide keyword searches, X accounts, Hacker News, Mastodon and RSS. New posts are indexed as they arrive, so the Dataset only holds what the user's Watchers have picked up since they were created.
   - `get_dataset_summary`: record count, last ingest time, busiest channels, and today's live-read usage. Call it first when you do not know what the Dataset holds.
   - `list_watchers`: which sources are being followed.
   - `search_dataset` (`query` required; `mode` "semantic" or "keyword"; `tier` "quick" or "deep"): search by meaning. Filters: `subreddits`, `platforms`, `any`, `include`, `exclude`, `tags`, `minScore`, `minComments`, `from`, `to`, `watcherId`, `limit` (max 40), `cursor`.
   - `list_records`: browse by filter without a search query. Same filters plus `view` ("all" or "matched") and `keyword`.
   - `get_record` (`id`): full body and tags for one record. List and search results trim long bodies.
   - `get_record_comments` (`id`, `limit`): the live comment thread for a Dataset record. It has its own daily comment-read cap.
2. **Live reads (metered, 1 credit each, daily cap).** Real-time reads of Reddit and Hacker News only.
   - `social_search` (`platform`, `query`, plus `subreddit`, `sort`, `time`, `limit` for Reddit; `sort`, `tags`, `minPoints`, `author`, `since` for Hacker News).
   - `social_list_posts` (`platform`, `channel` required for Reddit, `sort`, `time`, `limit`).
   - `social_get_post` (`platform`, `id`, `includeComments`, `commentLimit`, `commentOffset`).
   - `social_get_user`, `social_get_user_activity` (`platform`, `username`, `kind`, `limit`).
   - `social_find_channels` (`platform`, `query` or `channel`, `include`).
   - `get_usage` shows today's count, the cap and the reset time.
3. **Catalog (free, not metered).** Prowlo's directory of about 1,200 tracked subreddits with size, activity, growth and promo tolerance: `catalog_search_subreddits`, `catalog_get_subreddit`, `catalog_top_subreddits`.

There is also `research_get`, which reads a community research report the user has already run in Prowlo. If the user asks which subreddits suit their product, check it before rebuilding the answer from live reads.

## How to work

- **Start with the Dataset and the catalog.** They are free. Use live reads when the Dataset has no coverage for the question (no Watcher on that source, or the posts are older than the Watcher), or when the user needs the current state of a thread.
- **Keep live reads small.** Each call costs one credit from a daily cap. Prefer one well-aimed `social_search` over many. If a task would need more than about five live reads, say so and ask first. If `get_usage` shows the cap is close, tell the user.
- **Say where the answer came from.** "From your Dataset (r/saas Watcher, last 30 days)" and "from a live Reddit search" mean different things. Empty Dataset results usually mean no Watcher covers that source, not that nobody talks about it.
- **Cite links.** Every claim about what people said should point to the post or comment it came from, using the permalink or URL in the record. Quote short phrases, not whole posts. Give dates.
- **Do not overstate.** A handful of posts is anecdote, not a trend. Give counts when you have them. For counts over time, a Trend (`trend_list`, `trend_get`) is better than counting search results by hand.
- **Only read unless asked.** Creating or changing Watchers, Keyword Monitors, Alerts, Trends, tagging rules or the product profile changes the user's account. Do it only when the user asks for it, and confirm the exact settings before the call. Suggesting a Watcher is fine; creating one without being asked is not.

## Limits to keep in mind

- Prowlo is read-only toward Reddit and X. It cannot post, comment, vote or send messages. Do not offer to do any of these.
- Live reads cover Reddit and Hacker News. X, Mastodon and RSS are available only through Watchers and the Dataset.
- The `intent` filter is not available yet and returns no matches. Do not use it.
- Reddit results from live reads may show score and comment count as 0. Fetch the post if those numbers matter.
