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

Routines execute in Anthropic's cloud sandbox, so you do **not** need to
install anything locally to make the daily digest run. Pick the path that
matches where you use Claude Code.

### Path A — iPhone / web Claude Code app (zero install)

1. Open the `NWT77/pixel-assault` repo on the
   `claude/news-aggregator-automation-F2eWq` branch in the app.
2. Run `/schedule` in a session. When prompted:
   - **Prompt**: *"run the digest defined in `digest/prompt.md`"*
   - **Cadence**: `daily at 08:00` (your local timezone)
   - **Model**: `Haiku 4.5`
   - **Repo access**: enable write, so the routine can commit to `digest/archive/`
3. (Optional) Edit `digest/prompt.md` in the app, replacing `REPLACE_ME`
   YouTube channel IDs with real ones. Find a channel ID from the page
   source of any channel page — search for `channelId` or `"externalId"`.

That's it. Anthropic's sandbox reads `.mcp.json`, launches
`mcp-server-fetch`, runs the prompt, commits the output. Your phone can be
off when it fires.

### Path B — MacBook / desktop CLI (needed only for dry-runs)

Use this if you want to test the prompt on demand before trusting the
daily schedule, or iterate on `digest/prompt.md` faster than once a day.

1. Install `uv` so the `fetch` MCP server can launch locally:
   ```bash
   curl -LsSf https://astral.sh/uv/install.sh | sh
   ```
2. Open the repo:
   ```bash
   cd pixel-assault
   claude
   ```
   Approve the `fetch` MCP server on first prompt.
3. Schedule the routine with `/schedule` (same inputs as Path A), **or**
   do a one-shot dry run (see below).

## Dry-run the digest once

From a desktop Claude Code session in the repo, ask:

> run the digest defined in digest/prompt.md, but write the output to
> `digest/archive/dryrun.md` instead of today's date, and do not commit

This exercises every source end-to-end and lets you eyeball the output
before committing to the daily schedule. Delete `digest/archive/dryrun.md`
when satisfied.

Iterate on `digest/prompt.md`, re-run the dry-run, and only then register
the daily routine.

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
