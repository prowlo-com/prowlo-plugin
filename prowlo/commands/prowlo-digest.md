---
description: Summarize what a community or topic discussed recently. Use when you want a digest of a subreddit, a niche, or your Prowlo Watchers over the last few days.
argument-hint: "[topic or r/subreddit] [--days N]"
---

# /prowlo-digest

Write a digest of what was discussed in a community or about a topic, following the `niche-digest` skill.

Arguments: `$ARGUMENTS`

## Parse the arguments

- `r/<name>` or a bare subreddit name: the community to cover.
- Any other text: a topic, used as a semantic search query.
- `--days N`: the period. Default 7. Maximum 30.
- No arguments: digest everything the user's Watchers collected in the period.

State the resolved scope in one line before you start.

## Steps

1. `list_watchers` to see whether a Watcher covers the community or topic.
2. Gather posts from the Dataset (free):
   - Subreddit: `list_records` with `subreddits: ["<name>"]` and `from` set to N days ago.
   - Topic: `search_dataset` with `query` set to the topic and `from` set to N days ago.
   - No arguments: `list_records` with `from` set to N days ago, paged with `cursor`.
3. If the Dataset has no coverage, say so and use at most three live reads (1 credit each): `social_list_posts` (`platform` "reddit", `channel`, `sort` "top", `time` "week" or "day") for a subreddit, or `social_search` (`platform` "reddit", `query`, `time`) for a topic.
4. Open the few threads that carry each theme with `get_record` or `social_get_post`.
5. Write the digest in the `niche-digest` format: themes, notable threads with links, questions people asked, and what is worth watching.

## Writes

This command only reads. If you suggest a Watcher for a community that had no coverage, ask first and call `watcher_create` only after the user agrees.
