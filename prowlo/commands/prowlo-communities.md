---
description: Find and vet subreddits for an audience or problem. Use when you want to know where a group of people talks on Reddit and which communities are active and growing.
argument-hint: "<audience or problem>"
---

# /prowlo-communities

Find subreddits that match an audience or problem and rank them, following the `community-discovery` skill.

Arguments: `$ARGUMENTS`

If no audience or problem is given, ask for one in a single line and stop.

## Steps

1. `catalog_search_subreddits` with `query` set to the audience or problem written as a sentence. Run one or two more phrasings if results are thin. Free.
2. `catalog_get_subreddit` (`name`, `view` "stats") for the top 5 to 8 candidates. Free.
3. Optionally `catalog_get_subreddit` with `view` "similar" on the best pick to find neighbors, and `catalog_top_subreddits` to compare growth. Free.
4. Only if a candidate is outside the catalog, or the user asks for its rules, use `social_find_channels` (`platform` "reddit", `channel`, `include: ["rules"]`). 1 credit each.
5. Show a ranked table: subreddit, subscribers, posts per day, 30-day growth, promo tolerance, fit. Mark null values as unknown.

## Writes

This command only reads. After the table, offer to watch the communities the user picks. Call `watcher_create` (one `sources` entry per subreddit, `platform` "reddit") only after the user names which ones.
