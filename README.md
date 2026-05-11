# Ron's Claude Code Routines (Demo Repo)

Three Claude Code Routines running on autopilot for the Community Builders business. This is the repo Ron points his Routines at. Used to film the "Claude Code Routines: 3 Use Cases Every Coach Should Steal" YouTube video.

## What's running

### Routine 1: Daily Topic Scan
Every morning at 7am, scans YouTube + X/Twitter + GitHub + web for what's hot in Ron's niche (Skool, AI agents, coaching business). Synthesizes a ranked list of content ideas. Drops a Slack message.

### Routine 2: Pre-Call Sales Brief
Every weekday at 7am, pulls today's sales calls from the GHL calendar. For each booked contact, pulls tags + intake form + conversation history and writes a one-page brief: who they are, what they want, what to lead with. Posts to Slack so Ron walks into every call warm instead of cold.

### Routine 3: Fathom Calls into Short-Form Scripts
Wednesdays and Thursdays at 5pm (after Laser Coaching + AI Mastermind), pulls the latest Fathom recording, finds the 5 strongest moments, drafts 5 ready-to-film short-form scripts in Ron's voice, posts them all to Slack.

## Repo structure

```
claude-code-routines-demo/
├── .claude/
│   └── skills/                    # Skills the Routines call
│       ├── daily-topic-scan-cloud/
│       ├── pre-call-brief/
│       └── fathom-to-shorts/
├── reference/                     # Ron's voice + niche config
│   ├── niche.md                   # Ron's niche, audience, topic areas
│   ├── voice.md                   # Ron's tone rules, signature phrases
│   └── short-form-tone.md         # Tone rules for short-form scripts
├── routines/                      # Routine prompts (copy-paste into dashboard)
│   ├── 01-daily-topic-scan.md
│   ├── 02-pre-call-brief.md
│   └── 03-fathom-to-shorts.md
├── .env.example                   # Env vars needed
└── .gitignore
```

## Connectors needed in claude.ai

Authorized via OAuth (no manual API keys):
- **Slack** (notifications + brief + script output)
- **Fathom** (coaching call transcripts)

GHL uses a Private Integration Token (Bearer auth), passed as env var.

## Env vars (from .env.example)

| Name | Used by | Source |
|---|---|---|
| `YOUTUBE_API_KEY` | Routine 1 | Google Cloud Console |
| `XAI_API_KEY` | Routine 1 | xAI dashboard (optional, skip if unused) |
| `GHL_API_KEY` | Routine 2 | GHL → Settings → Private Integrations |
| `GHL_LOCATION_ID` | Routine 2 | Your GHL sub-account location ID |
| `SALES_CALENDAR_IDS` | Routine 2 | Comma-separated GHL calendar IDs for sales calls |

Routine 3 needs no env vars (pure MCP).

## Schedules

| Routine | Cron | Local time |
|---|---|---|
| Daily Topic Scan | `0 7 * * *` | Every day at 7am ET |
| Pre-Call Sales Brief | `0 7 * * 1-5` | Weekdays at 7am ET |
| Fathom to Shorts | `0 17 * * 3,4` | Wed/Thu at 5pm ET |

## How this connects to the video

The YouTube video "Claude Code Routines: 3 Use Cases Every Coach Should Steal" demos all three of these. Setup walkthrough is filmed live, then the video shows real Slack output from a week of these running.
