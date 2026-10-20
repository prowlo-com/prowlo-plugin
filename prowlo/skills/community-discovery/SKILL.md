---
name: community-discovery
description: Use when the user wants to find or compare subreddits for an audience, niche or problem. For example, "where do freelance designers hang out on Reddit", "which subreddits discuss home lab hardware", "is r/startups active", "find communities similar to r/saas", or "which subreddits in this space are growing".
---

# Community discovery

Find subreddits that match an audience or problem, check whether they are worth following, and offer to watch the good ones. The catalog tools are free, so do most of the work there. Follow the `social-research` skill for tool choice and costs.

## Steps

1. **Check for a research report.** If the user is asking about communities for their own product, call `research_get` (default `section` "summary"). It returns a report if Prowlo has prepared one for your workspace. If one exists, build on it and say so.
2. **Search the catalog.** Call `catalog_search_subreddits` with a `query` that describes the audience or problem in a sentence (for example "freelance designers looking for clients and pricing advice"), not one keyword. Try two or three phrasings if the first results are thin.
3. **Vet the candidates.** For the top 5 to 8, call `catalog_get_subreddit` with `name` (default `view` "stats"). Look at:
   - `subscribers` for size.
   - `postsPerDay` and `lastPostAt` for activity. A big subreddit with few posts a day may be quiet.
   - `distinctPosterRatio` for how many different people post.
   - `growth7d` and `growth30d`. Null means not enough tracking history, not zero growth.
   - `promoTolerance` (low, medium, high): how strict the community's rules on self-promotion are. Null means not profiled yet, not "no rules".
4. **Expand if useful.** `catalog_get_subreddit` with `view` "similar" lists neighbors by content and audience. `catalog_top_subreddits` (`metric` "largest", "most-active" or "fastest-growing"; optional `size` band) helps benchmark a candidate against the field.
5. **Go live only when needed.** If a subreddit is not in the catalog, or the user needs its current rules, use `social_find_channels` with `platform` "reddit" and `channel` (add `include: ["rules"]` for rules). This costs one credit.

## Output format

A short ranked table: subreddit, subscribers, posts per day, 30-day growth, posting rules, and one line on fit. Then two or three sentences on the best picks and any trade-offs (for example "large but mostly memes" or "small and very active"). Flag null values as unknown instead of guessing.

## Offering to watch

After the table, offer to create Watchers for the picks the user likes, so new posts flow into their Dataset. Do not create any until the user says which ones. Then call `watcher_create` with one entry per subreddit, for example `sources: [{ "channel": "freelance", "platform": "reddit" }, { "channel": "graphic_design", "platform": "reddit" }]`, plus `tags` if the user wants them. Watchers count against plan limits; if the call fails on a limit, say so.

## Boundaries

Treat this as research into where an audience talks. Do not plan promotional posting campaigns, suggest ways around subreddit rules, or recommend posting as if unaffiliated.
