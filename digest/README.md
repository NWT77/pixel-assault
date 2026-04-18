# Claude Daily Digest

Daily top-10 news digest for the keyword "Claude", focused on algo-trading
use cases and new Claude / Anthropic features.

## Architecture

| Layer | Choice | Why |
|---|---|---|
| Scheduler | [Claude Code Routines](https://code.claude.com/docs/en/scheduled-tasks) | Anthropic-hosted cron. Runs on their infra; your machine can be off. |
| Agent | The routine prompt itself | No Agent SDK wrapper needed for a single-shot daily task. |
| Content | Single `fetch` MCP server | Hits public free endpoints directly — no per-call cost. |
| Sources | HN Algolia, Reddit JSON, Google News RSS, Anthropic blog, YouTube RSS | All free, no API keys. |
| Output | `digest/archive/YYYY-MM-DD.md`, committed to this branch | Version-controlled history. |

**Cost**: $0 on a Claude Max subscription (Routines runs count against plan usage).

## Setup

### 1. Install `uv` (for the `fetch` MCP server)

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

The `.mcp.json` at the repo root tells Claude Code to launch
`mcp-server-fetch` via `uvx` on demand. No other install step needed.

### 2. Open the repo in Claude Code

```bash
cd pixel-assault
claude
```

First launch will prompt you to approve the `fetch` MCP server. Approve it.

### 3. Schedule the routine

Inside Claude Code:

```
/schedule
```

When prompted:
- **Prompt**: paste the contents of `digest/prompt.md`, or reference it: *"run the digest defined in digest/prompt.md"*
- **Cadence**: `daily at 08:00` (your local timezone)
- **Model**: `Haiku 4.5` — cheapest and plenty for this task
- **Repo access**: enable, so the routine can write + commit to `digest/archive/`

### 4. (Optional) Add YouTube channels

Edit `digest/prompt.md`, replacing `REPLACE_ME` with real channel IDs.
Find a channel ID from the page source of any channel page — search for
`channelId` or `"externalId"`.

## Output

Each run produces `digest/archive/YYYY-MM-DD.md` with the top 10 items,
categorized **trading** / **feature** / **other**, with 2–3 sentence summaries
and source links.

## Tuning

- **More trading, less features**: edit the target mix in step 6 of `prompt.md`.
- **New sources**: add URLs under the relevant section in `prompt.md`.
- **Different hour**: re-run `/schedule` and pick a new time.
- **Pause**: `/schedule list` then remove the routine.

## Why not the `web_search` tool?

`web_search` is $0.01 per call. For a daily digest pulling from ~15 sources,
that's $4–5/month — cheap, but the free endpoints above cover the same ground
(and Reddit + HN are arguably higher-signal than generic web search for this
topic). If you ever want broader coverage, add `web_search` to the prompt's
tool list.
