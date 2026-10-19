# Prowlo for Claude

Prowlo lets Claude search and monitor public discussions on Reddit, Hacker News, X, Mastodon and RSS feeds. You create Watchers that follow sources you choose. New posts from those sources are indexed into your own Dataset, which Claude can search by meaning. Claude can also read Reddit and Hacker News live, look up subreddits in a free catalog, and set up Alerts and Trends when you ask.

Prowlo is read-only toward Reddit and X. It never posts, comments, votes or sends messages.

## What this plugin does

This plugin adds three things to Claude:

1. **A connection to the Prowlo MCP server** at `https://api.prowlo.com/mcp`. This is a remote, hosted service run by Prowlo. The plugin does not run any code on your computer.
2. **Four skills** that tell Claude how to use the Prowlo tools well.
3. **Three slash commands** for common research tasks.

### What data is sent to Prowlo

When Claude calls a Prowlo tool, the tool name and its arguments are sent to `https://api.prowlo.com/mcp` under your Prowlo account. Arguments are things like search queries, subreddit names, brand names, date ranges, and the settings of Watchers or Alerts you ask Claude to create. Prowlo returns posts, records and statistics, which Claude then reads. Your conversation with Claude is not sent to Prowlo, only these tool calls. See the [privacy policy](https://prowlo.com/privacy) for how Prowlo handles account data.

### Sign-in

The server uses OAuth. The first time Claude uses a Prowlo tool you will be asked to sign in to Prowlo (or create an account) and approve access. No API key is needed. You can revoke access at any time from your Prowlo account settings.

## Install

In Claude Code:

```
/plugin install prowlo --marketplace prowlo-com/prowlo-plugin
```

Or add the marketplace first and then install:

```
/plugin marketplace add prowlo-com/prowlo-plugin
/plugin install prowlo@prowlo
```

In the Claude apps, install Prowlo from the plugin directory and sign in when prompted.

## First prompts

- "What does my Prowlo Dataset hold right now, and which Watchers feed it?"
- "Find subreddits where freelance designers talk about pricing, and rank them by activity and growth."
- "What did r/selfhosted discuss this week? Give me the main themes and the threads worth reading."
- "Search my Dataset for posts where people complain about onboarding in project management tools, and link the best examples."
- "Are people mentioning Plausible or Fathom on Reddit this month? Summarize what they like and dislike."

## Skills

- **social-research**: how to choose between the Dataset, live reads and the catalog, how to keep metered reads low, and how to cite post links in answers.
- **niche-digest**: a daily or weekly digest of what a community or topic discussed, with themes, notable threads and the questions people asked.
- **brand-and-competitor-monitoring**: finds mentions of a brand and its competitors and summarizes sentiment. Sets up Keyword Monitors, Watchers or Alerts only when you ask.
- **community-discovery**: finds and vets subreddits by size, activity, growth and promo tolerance, then offers to watch the ones you pick.

## Commands

- `/prowlo-digest [topic or r/subreddit] [--days N]`: digest of a community, a topic, or all your Watchers.
- `/prowlo-mentions <brand> [competitors...]`: recent mentions and a sentiment summary.
- `/prowlo-communities <audience or problem>`: ranked list of matching subreddits.

The commands only read data. When a command suggests creating a Watcher, Keyword Monitor or Alert, Claude asks you first.

## Tools

The Prowlo MCP server provides these tools. Read tools are marked read-only to Claude; tools that change your Prowlo account are not.

**Dataset** (your indexed posts; free, rate-limited)
`get_dataset_summary`, `search_dataset`, `list_records`, `get_record`, `get_record_comments` (live comment thread for a record, with its own daily cap), `tag_record`

**Live reads** (Reddit and Hacker News in real time; 1 credit each, daily cap)
`social_search`, `social_list_posts`, `social_get_post`, `social_get_user`, `social_get_user_activity`, `social_find_channels`

**Subreddit catalog** (about 1,200 tracked subreddits; free)
`catalog_search_subreddits`, `catalog_get_subreddit`, `catalog_top_subreddits`

**Watchers and Keyword Monitors**
`list_watchers`, `watcher_create`, `watcher_update`, `watcher_delete`, `watcher_remove_source`, `keyword_monitor_list`, `keyword_monitor_create`

**Alerts** (Slack or webhook notifications)
`alert_list`, `alert_create`, `alert_update`, `alert_delete`, `alert_get_deliveries`

**Trends** (mention counts over time)
`trend_list`, `trend_get`, `trend_create`, `trend_update`, `trend_delete`

**Tagging rules**
`tag_rule_list`, `tag_rule_create`, `tag_rule_update`, `tag_rule_delete`

**Product profile and research**
`get_product_profile`, `update_product_profile`, `research_get`

**Account and status**
`get_account`, `get_usage`, `get_crawl_activity`, `get_operational_alerts`

## Safety

- **Read-only toward Reddit and X.** Prowlo has no tools that post, comment, vote or send messages, and this plugin does not ask Claude to do any of these.
- **No Reddit or X credentials.** Claude never sees or needs your Reddit or X login.
- **Writes only change your Prowlo account.** Creating or deleting Watchers, Alerts, Trends or tagging rules affects your Prowlo workspace only. The skills and commands tell Claude to ask before making these changes.
- **Metered reads are kept small.** The skills tell Claude to use the free Dataset and catalog first and to ask before running many live reads.

## Pricing

Prowlo has a free 14-day trial, then costs $19 per month. A Prowlo account is required. Sign up at [prowlo.com](https://prowlo.com).

## Privacy and support

- Documentation: [prowlo.com/docs/mcp-setup](https://prowlo.com/docs/mcp-setup)
- Privacy policy: [prowlo.com/privacy](https://prowlo.com/privacy)
- Terms of service: [prowlo.com/terms](https://prowlo.com/terms)
- Support: [prowlo.com/contact](https://prowlo.com/contact) or support@prowlo.com

## License

The files in this plugin (skills, commands, manifest) are released under the MIT License. See [LICENSE](LICENSE). The Prowlo service at `api.prowlo.com` is a separate hosted product covered by the Prowlo [terms of service](https://prowlo.com/terms).
