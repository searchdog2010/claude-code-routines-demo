# Routine 1: Daily Topic Scan

Pulls top content ideas from YouTube + X/Twitter + GitHub + Web every morning at 7am, synthesizes a ranked report, posts to Slack.

---

## Step-by-step setup

### Step 1: Authorize MCP connectors (one-time)

Go to **claude.ai → Settings → Connectors** and authorize:
- **Slack** — for posting the morning report

If already authorized, skip.

### Step 2: Get API keys

You need two API keys for this Routine:

**YouTube Data API v3:**
1. Go to console.cloud.google.com
2. Create a project (or use an existing one)
3. Enable "YouTube Data API v3"
4. Credentials → Create Credentials → API Key
5. Copy the key

**xAI / Grok API (optional, skip if you don't have one):**
1. Go to console.x.ai
2. Sign up, fund $5 of credits
3. Create an API key
4. Copy the key

### Step 3: Create the Routine

1. Go to **claude.ai/code/routines**
2. Click **Create Routine**
3. Name it: `Daily Topic Scan`
4. Point it at this repo: `searchdog2010/claude-code-routines-demo` (or your fork)
5. Paste the Routine prompt (below) into the prompt field
6. Add the env vars (below)
7. Set the schedule: `0 7 * * *` with timezone `America/New_York`
8. Click **Save**

### Step 4: First run

1. Click **Run Now** to test before waiting for tomorrow's 7am fire
2. Wait ~2-3 minutes
3. Check the #content Slack channel for the morning summary
4. If anything looks off, check the Routine logs in the dashboard

---

## Routine prompt (copy-paste)

```
Run the daily-topic-scan-cloud skill in this repo.

Read reference/niche.md and reference/voice.md first for context.

Pull from 4 sources in parallel:
- YouTube Data API (use YOUTUBE_API_KEY env var): 4 searches based on my niche topics, top 10 results each, last 7 days
- X/Twitter via Grok (use XAI_API_KEY env var, skip if not set): top 5 trending posts in my niche, last 24 hours
- GitHub trending (no key): WebFetch github.com/trending?since=daily, pull top 5 AI/automation repos
- WebSearch: 5 queries based on my niche topics, read top 3 results each

Synthesize signals into the top 5 ranked content ideas for today. For each:
- Topic in 1 sentence
- Why-now (specific signal referenced)
- My angle (how I would uniquely take this)
- Format suggestion (Reel / YouTube Short / long-form / community post / email)
- Source link

Apply quality filters: cut anything <5K total views, cut anything covered to death in last 30 days, boost anything appearing in 2+ sources.

Post a summary to Slack via the Slack MCP. Channel: #content. Format:

📡 *Daily Topic Scan — [Date]*

*Top 3 ideas today:*
1. [Idea 1] - [one-line why]
2. [Idea 2] - [one-line why]
3. [Idea 3] - [one-line why]

*Sources scanned:* YouTube (X), X/Twitter (X), GitHub (X), Web (X)

*See top 5 in full:*
[paste all 5 ideas with details inline]

If the skill fails at any step, post the error to the same Slack channel with ⚠️ prefix. Don't crash.
```

## Env vars

| Name | Value |
|---|---|
| `YOUTUBE_API_KEY` | Your YouTube Data API v3 key |
| `XAI_API_KEY` | Your xAI/Grok key (optional) |

Slack handled via MCP. No webhook URL needed.

## Schedule

`0 7 * * *` (every day at 7am)

Set Routine timezone to `America/New_York` if available, otherwise convert to UTC (`0 12 * * *` for 7am ET during standard time, `0 11 * * *` during DST).

## What you get

A Slack message at 7am with:
- Top 3 content ideas ranked
- Signal counts per source
- Full top 5 with rationale, angles, format suggestions, and source links

Cost: ~1 Routine run per day. Takes 2-3 minutes per run.
