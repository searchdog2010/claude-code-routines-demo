---
name: daily-topic-scan-cloud
description: "Cloud-runnable daily content topic scan. Hits YouTube Data API, X via Grok API, GitHub trending, and WebSearch to surface high-leverage content ideas in the coach's niche. Synthesizes into a ranked report and posts to Slack. Cloud-only version: no Skool/Chrome dependencies. Use when running as a Claude Code Routine on a daily schedule."
---

> **ABSOLUTE RULE: NO DASHES FOR PUNCTUATION.** Never use em dashes, en dashes, or double hyphens (--) as punctuation in output. Use commas, periods, or rewrite.

# Daily Topic Scan (Cloud)

Runs every morning as a Claude Code Routine. Surfaces the highest-leverage content ideas for the coach's niche from API sources only. No browser, no local files.

## Step 0: Load Context

Read these reference files first:
- `reference/niche.md` — Ron's niche, audience, topic areas, search terms
- `reference/voice.md` — Ron's voice (used in the synthesis section)

## Step 1: Pull from 4 sources in parallel

### Source A: YouTube Data API (4 searches)

Use the YOUTUBE_API_KEY env var. Search YouTube for trending videos in the niche.

Run 4 searches based on the niche file's "Search terms my audience would use" section. Examples:
- Search term 1 + "2026"
- Search term 2 + "2026"
- Search term 3 + "2026"
- Search term 4 + "2026"

For each, pull top 10 results sorted by view count, filtered to last 7 days. Capture: title, channel name, view count, channel subscriber count, video URL.

### Source B: X / Twitter via Grok API

Use XAI_API_KEY env var. Hit the Grok API to find trending posts in the niche.

Prompt to Grok: "What are the top 5 most-discussed posts on X in the last 24 hours about [niche topic areas from niche-example.md]. Return: post text, author, engagement count, post URL."

If XAI_API_KEY isn't set, skip this source and note "X source skipped (no XAI_API_KEY)" in the report.

### Source C: GitHub Trending

WebFetch `https://github.com/trending?since=daily` (no API key needed for the trending page).

Pull top 5 AI/automation repositories trending today. Capture: repo name, description, star count, language.

Filter to repos relevant to coaches and automation. Skip dev-only stuff (compiler internals, language runtimes, etc.) unless they're directly coach-relevant.

### Source D: WebSearch (5 queries)

Run 5 WebSearch queries using the WebSearch tool:
1. `[niche topic 1] news today 2026`
2. `[niche topic 2] news today 2026`
3. `[niche topic 1] OR [niche topic 2] trending site:x.com today`
4. `[niche topic 3] announcement today 2026`
5. `[primary niche topic] 2026 update`

Substitute the topics from the niche file. Read the top 3 results for each query.

## Step 2: Synthesize into a ranked report

Compile a single markdown report titled `Daily Topic Scan - YYYY-MM-DD`.

Structure:

```markdown
# Daily Topic Scan
**Date:** YYYY-MM-DD

## Top 5 Content Ideas (Ranked)

For each idea (1-5):
- **Topic:** 1-sentence description
- **Why it matters now:** 1-2 sentences. Reference the signal that triggered it (which source, which specific data point).
- **Angle:** the coach's version. How would THIS coach approach this topic, given their positioning?
- **Format suggestion:** Reel / YouTube Short / long-form / community post / email
- **Source:** link

## Signals scanned
- YouTube: X videos across 4 searches
- X/Twitter: X posts
- GitHub trending: X repos relevant
- Web: X searches, X results read

## Cold takes (skipped but noted)
List 2-3 topics that came up but didn't make the cut, and why. Helps the coach see what's NOT working.
```

## Step 3: Quality rules

Before finalizing the ranked list, apply these filters:

**Cut a topic if:**
- It has fewer than 5K total views across all signals
- It's been covered to death in the coach's niche in the last 30 days
- It's only on one source (single signal is weak)
- It's not relevant to the audience defined in niche-example.md

**Boost a topic if:**
- It appears in 2+ sources (YouTube + X, GitHub + Web, etc.)
- It has a contrarian angle the coach could uniquely take
- It connects to a specific number, story, or proof point
- It pairs with a moment the coach has been talking about recently

## Step 4: Post summary to Slack

Use the Slack MCP (channel: #content-reports) env var. Format the message:

```
📡 *Daily Topic Scan — [Date]*

*Top 3 ideas today:*
1. [Idea 1 title] - [one-line why]
2. [Idea 2 title] - [one-line why]
3. [Idea 3 title] - [one-line why]

*Sources scanned:* YouTube (X), X/Twitter (X), GitHub (X), Web (X)
*Full report:* [link to the markdown file in the repo if saved, or paste the top 5 inline]
```

Keep it under 10 lines. Slack markdown: `*bold*`, `_italic_`, `<URL|text>`.

## Step 5: Error handling

If the skill fails at any step:
1. Capture the error message
2. Slack the same webhook with: `⚠️ Daily Topic Scan failed: [step that failed]. Error: [first 200 chars of error]. Will retry tomorrow.`
3. Exit gracefully

Don't crash, don't retry mid-run. Daily cadence means tomorrow's run is the safety net.

## Required env vars

- `YOUTUBE_API_KEY` — YouTube Data API v3 key (get from Google Cloud Console)
- `XAI_API_KEY` — Grok / xAI API key (optional, will skip X source if not set)
- `Slack MCP (channel: #content-reports)` — Slack incoming webhook for the channel to post to

## Schedule recommendation

`0 7 * * *` — every day at 7am (coach's local timezone, set via Routine timezone field if available)
