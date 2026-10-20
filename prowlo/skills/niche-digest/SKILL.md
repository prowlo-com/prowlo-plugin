---
name: niche-digest
description: Use when the user wants a summary of what a community or niche has been talking about recently. For example, "what happened in r/selfhosted this week", "give me a daily digest of my Watchers", "what are indie hackers discussing lately", or "what questions keep coming up about home espresso".
---

# Niche digest

A digest tells the user, in a few minutes of reading, what a community discussed over a period: the main themes, the threads worth opening, and the questions people keep asking. Follow the `social-research` skill for tool choice and costs.

## Steps

1. **Pin down scope.** Work out the community or topic and the period. Default to the last 7 days. If the user named a subreddit, use it. If they named a topic, treat it as a semantic query.
2. **Check coverage.** Call `list_watchers` (or `get_dataset_summary`) to see whether a Watcher already covers the community or topic.
3. **Gather posts.**
   - If a Watcher covers it: use `list_records` with `subreddits` (or `watcherId`), `from` set to the start of the period, and a sensible `limit`. Page with `cursor` if needed. For a topic, use `search_dataset` with the topic as `query` and the same date filter.
   - If nothing covers it: say so, then use a small number of live reads. For a subreddit, `social_list_posts` with `platform` "reddit", `channel`, `sort` "top" and `time` "week" (or "day"). For a topic, `social_search` with `time` set to match the period. For Hacker News, `social_search` with `platform` "hackernews" and `since`.
4. **Read the important threads.** For the few threads that drive a theme, open them with `get_record` (Dataset) or `social_get_post` (live). Read comments only where the discussion is the point, and keep it to a handful of threads. `get_record_comments` and `social_get_post` both count against daily caps.
5. **Write the digest.**

## Output format

- **Header:** community or topic, period, source (Dataset or live), number of posts reviewed.
- **Themes (3 to 6):** one or two sentences each, with how many posts touched it and links to the best examples.
- **Notable threads (up to 8):** title as a link, date, score and comment count when known, and one line on why it matters.
- **Questions people asked:** recurring questions in the users' own words, with links. These are often the most useful part.
- **Worth watching:** anything new or growing, stated with care if the evidence is thin.

Keep it skimmable. Link every claim to a post.

## After the digest

If the digest relied on live reads, you may mention once that a Watcher on this community would collect posts going forward and make future digests free. Create it only if the user says yes, and confirm the source first (`watcher_create` with `sources: [{ "channel": "<name>", "platform": "reddit" }]`).
