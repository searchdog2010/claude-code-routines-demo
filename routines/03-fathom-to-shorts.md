# Routine 3: Fathom Calls into Short-Form Scripts

Pulls latest coaching call transcript every Wednesday + Thursday at 5pm. Finds 5 standout moments. Drafts 5 ready-to-film short-form scripts in your voice. Posts them all to Slack.

The simplest of the three Routines. No env vars needed.

---

## Step-by-step setup

### Step 1: Authorize MCP connectors (one-time)

Go to **claude.ai → Settings → Connectors** and authorize:
- **Fathom** — for pulling call transcripts
- **Slack** — for posting the scripts

If already authorized, skip.

### Step 2: Create the Routine

1. Go to **claude.ai/code/routines**
2. Click **Create Routine**
3. Name it: `Fathom to Short-Form Scripts`
4. Point it at this repo
5. Paste the Routine prompt (below)
6. No env vars needed
7. Set the schedule: `0 17 * * 3,4` with timezone `America/New_York`
8. Click **Save**

### Step 3: First run

1. Click **Run Now** (assuming you've done a Laser Coaching or AI Mastermind call recently)
2. Wait ~3-5 minutes
3. Check the #content Slack channel
4. You should see one message with all 5 scripts inline (hook, body, CTA, source quote, timestamp for each)
5. Copy any one out when you're ready to film

If no matching call exists in the last 48 hours, the Routine will Slack you "no new coaching call found, skipping" and exit cleanly. Not an error.

---

## Routine prompt (copy-paste)

```
Fathom into Short-Form Scripts. Be strictly directive. Run in exact order. Stop the moment an exit condition fires.

STEP 1: Use the Fathom MCP list_meetings tool. Filter to last 48 hours.

STEP 2: EARLY EXIT CHECK. Look at the returned meetings. Find ones whose title contains "Laser Coaching" OR "AI Mastermind".
  - If ZERO matches: Post via Slack MCP to channel #content: "🎬 No new coaching call found in the last 48 hours (Laser Coaching or AI Mastermind). Skipping today's run."
  - STOP. Return success. Do not query anything else. Do not retry. Do not search older meetings.

STEP 3: If a match exists, pull the full transcript via Fathom MCP get_meeting_transcript using the most recent matching recording_id.

Read the entire transcript. Find 5 standout moments matching patterns A-E from the skill:
- A. Member question + my answer
- B. Framework / teaching moment with structure
- C. Story with specific numbers or names
- D. Contrarian take
- E. Breakthrough moment

Apply the quality gate (4 questions). Cut anything generic. 3 strong scripts beat 5 weak ones.

For each surviving moment, write a 30-60 sec short-form script in my voice (use voice.md):
- Hook (5-7 sec): stops scrolling, specific, direct
- Body (20-40 sec): the insight using my actual words from the transcript where possible
- CTA (3-5 sec): one specific next step
- Source quote: exact text from transcript
- Timestamp: mm:ss if available
- Moment type: A/B/C/D/E

Post all scripts to Slack via Slack MCP. Channel: #content. Format:

🎬 *5 New Short-Form Scripts*
*Source:* [Call Type] - [Date]

*Script 1: [Working title]*
> Hook: [hook]
> Body: [body]
> CTA: [CTA]
_Source quote:_ "[exact text]"
_Timestamp:_ [mm:ss] | _Type:_ [A/B/C/D/E]

---

(repeat for each script)

Quality rules:
- Pull script content from the transcript, do not invent
- Vary the 5 scripts in format (story, take, list, contrarian, framework)
- Skip moments where I was uncertain or correcting an earlier statement
- If only 3 or 4 moments are strong enough, generate that many and note the count
```

## Env vars

**None.** Fathom + Slack both via MCP.

## Schedule

`0 17 * * 3,4` (Wednesday + Thursday at 5pm)

Matches your call schedule (Wed = Laser Coaching, Thu = AI Mastermind). Adjust the days if your schedule differs:
- Daily coaching calls: `0 17 * * 1-5`
- Monday only: `0 17 * * 1`
- Tue/Thu: `0 17 * * 2,4`

## What you get

A Slack message at 5pm on call days with all 5 short-form scripts inline. Each has the hook, body, CTA, source quote so you can verify the script reflects what you actually said. Batch-film Thursday or Friday.

Cost: ~2 Routine runs per week. Takes 3-5 minutes per run.
