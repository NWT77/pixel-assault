# Claude Daily Digest — Routine Prompt

You are producing today's "Claude" news digest. All sources below are free
public endpoints; call them via the `fetch` MCP server. Do not use the paid
`web_search` tool.

## Sources

### Hacker News (Algolia — free, no key)
- https://hn.algolia.com/api/v1/search?query=Claude+AI&tags=story&hitsPerPage=30
- https://hn.algolia.com/api/v1/search?query=Anthropic&tags=story&hitsPerPage=30

### Reddit (JSON — free, no key)
- https://www.reddit.com/r/ClaudeAI/new.json?limit=50
- https://www.reddit.com/r/Anthropic/new.json?limit=50
- https://www.reddit.com/search.json?q=Claude+AI+trading&sort=new&t=day&limit=25
- https://www.reddit.com/r/singularity/search.json?q=Claude&restrict_sr=1&sort=new&t=day&limit=25
- https://www.reddit.com/r/LocalLLaMA/search.json?q=Claude&restrict_sr=1&sort=new&t=day&limit=25
- https://www.reddit.com/r/algotrading/search.json?q=Claude&restrict_sr=1&sort=new&t=week&limit=25

### Google News RSS (free, no key)
- https://news.google.com/rss/search?q=%22Claude%22+Anthropic&hl=en-US&gl=US&ceid=US:en
- https://news.google.com/rss/search?q=Claude+AI+trading&hl=en-US&gl=US&ceid=US:en
- https://news.google.com/rss/search?q=Anthropic+release&hl=en-US&gl=US&ceid=US:en

### Anthropic official
- https://www.anthropic.com/news
- https://www.anthropic.com/engineering

### YouTube (RSS, free, no key) — add channel IDs below
Find a channel's ID at https://www.youtube.com/account_advanced (for your own)
or via the page source of any channel page (search for "channelId").
- Anthropic: https://www.youtube.com/feeds/videos.xml?channel_id=REPLACE_ME
- (add more: AI Explained, Matt Berman, AI Jason, etc.)

## Task

1. Fetch every source above in parallel.
2. Parse JSON / RSS / HTML and normalize into `{title, url, source, published_at, blurb}`.
3. Drop items older than 30 hours.
4. Deduplicate by URL and near-duplicate title.
5. Classify each item into one of:
   - **trading** — algo trading, finance, quant, market-related uses of Claude
   - **feature** — new Claude model, API, Claude Code, skill, MCP, or product feature
   - **other** — community commentary, tutorials, benchmarks, opinion
6. Rank. Target mix: ~4 trading, ~5 feature, ~1 other — flex based on the day.
   Prefer official Anthropic sources and high-signal posts (HN points, Reddit
   upvotes, reputable outlets) over low-engagement noise.
7. Pick the **top 10** overall.
8. Summarize each in 2–3 sentences: what happened, why it matters, and a
   one-line "so what" for a trader or Claude power user.
9. Write to `digest/archive/YYYY-MM-DD.md` (UTC date).
10. `git add`, commit with message `digest: YYYY-MM-DD`, and `git push`.

## Output format

```markdown
# Claude Daily Digest — YYYY-MM-DD

**Trading:** N items · **Features:** N items · **Other:** N items

## 1. <Headline>
**Source:** <source name> · **Category:** trading|feature|other
**Link:** <url>

<2–3 sentence summary. Last sentence = why it matters for trading or Claude power users.>

## 2. <Headline>
...
```

## Guardrails

- If a source fails, skip it and note the failure at the bottom of the digest
  under `## Fetch errors`. Do not abort the run.
- Never hallucinate a link. If you cannot confirm a URL, drop the item.
- If fewer than 10 qualifying items exist, ship what you have and say so.
- Keep each summary under 60 words.
