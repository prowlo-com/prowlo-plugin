---
name: social-research
description: Use when the user wants to research what people say on Reddit, Hacker News and other public sources (Mastodon, RSS feeds, X) with Prowlo. For example, finding posts about a topic, checking what a community thinks of a product or problem, looking up a specific thread, or asking what their Prowlo Watchers have collected.
---

# Social research with Prowlo

Prowlo is a hosted MCP server. It gives Claude three kinds of social data. Pick tools by which one answers the question, and use the cheapest one that does.

## The three data planes

1. **Dataset (free, rate-limited).** The user's own indexed posts, collected over time by their Watchers. Each Watcher follows one source, set by its `platform` value:
   - `reddit`: a subreddit (the default). `channel` is the subreddit name without r/.
   - `reddit_search`: a Reddit-wide keyword search, run daily (the dashboard calls these Keyword Monitors). `channel` is the search query.
   - `hackernews`: a Hacker News search. `channel` is the search query.
   - `mastodon`: a hashtag on a Mastodon instance, for example `mastodon.social/#selfhosted`.
   - `rss`: an RSS or Atom feed. `channel` is the feed URL.
   - `x_user`: an X account's public posts (paid plan only).
   - `reddit_user`: a community's or company's public Reddit account.

   New posts are indexed as they arrive, so the Dataset only holds what the user's Watchers have picked up since they were created.
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
   - `social_get_user`, `social_get_user_activity` (`platform`, `username`, `kind`, `limit`). Use these only to judge an account in a thread the user is reading (for example account age). Don't build profiles of individuals or track specific people.
   - `social_find_channels` (`platform`, `query` or `channel`, `include`).
   - `get_usage` shows today's count, the cap and the reset time. The cap is 200 live reads a day (50/day during the trial).
3. **Catalog (free, not metered).** Prowlo's directory of about 1,200 tracked subreddits with size, activity, growth and posting rules: `catalog_search_subreddits`, `catalog_get_subreddit`, `catalog_top_subreddits`.

There is also `research_get`, which reads a community research report if Prowlo has prepared one for your workspace. If the user asks which subreddits suit their product, check it before rebuilding the answer from live reads.

## How to work

- **Start with the Dataset and the catalog.** They are free. Use live reads when the Dataset has no coverage for the question (no Watcher on that source, or the posts are older than the Watcher), or when the user needs the current state of a thread.
- **Keep live reads small.** Each call costs one credit from a daily cap. Prefer one well-aimed `social_search` over many. If a task would need more than about five live reads, say so and ask first. If `get_usage` shows the cap is close, tell the user.
- **Say where the answer came from.** "From your Dataset (r/saas Watcher, last 30 days)" and "from a live Reddit search" mean different things. Empty Dataset results usually mean no Watcher covers that source, not that nobody talks about it.
- **Cite links.** Every claim about what people said should point to the post or comment it came from, using the permalink or URL in the record. Quote short phrases, not whole posts. Give dates.
- **Do not overstate.** A handful of posts is anecdote, not a trend. Give counts when you have them. For counts over time, a Trend (`trend_list`, `trend_get`) is better than counting search results by hand.
- **Only read unless asked.** Creating or changing Watchers, Keyword Monitors, Alerts, Trends, tagging rules or the product profile changes the user's account. Do it only when the user asks for it, and confirm the exact settings before the call. Suggesting a Watcher is fine; creating one without being asked is not.

## Limits to keep in mind

- Prowlo is read-only toward every source. It cannot post, comment, vote or send messages. Do not offer to do any of these.
- Live reads cover Reddit and Hacker News. X, Mastodon and RSS are available only through Watchers and the Dataset.
- Reddit results from live reads may show score and comment count as 0. Fetch the post if those numbers matter.
