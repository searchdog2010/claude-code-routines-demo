---
name: fathom-to-shorts
description: "Turn the latest Fathom coaching call recording into 5 ready-to-film short-form scripts. Pulls the transcript via Fathom MCP, identifies the 5 strongest moments using the moment patterns from short-form-tone.md, drafts a 30-60 second script for each in the coach's voice, and saves everything to a Notion database. Runs as a Claude Code Routine on a schedule that matches the coach's recurring calls (typically Wed/Thu evening)."
---

> **ABSOLUTE RULE: NO DASHES FOR PUNCTUATION.** Never use em dashes, en dashes, or double hyphens (--) as punctuation in output. Use commas, periods, or rewrite.

# Fathom to Short-Form Scripts

Pulls today's coaching call transcript, finds 5 standout moments, drafts 5 ready-to-film short-form scripts, saves to Notion.

## Step 0: Load Context

Read these reference files first:
- `reference/voice.md` — used for script tone
- `reference/short-form-tone.md` — defines what a good short-form script looks like for this repo
- `reference/niche.md` — used to filter "what would my audience care about"

## Step 1: Pull the latest Fathom recording

Use the Fathom MCP (Connectors UI handles auth).

Tool: `list_meetings` with `limit: 10`.

Filter to find the most recent recording matching the call type filters (configurable in the Routine prompt):
- Default: title contains "Laser Coaching" OR "AI Mastermind"
- Coach can configure: title contains "[their call name]"

If multiple matches today, pick the most recent.

If zero matches in the last 48 hours: Slack `📞 Fathom-to-Shorts: no new coaching call found in the last 48 hours. Skipping today's run.` and exit cleanly. Not an error.

## Step 2: Pull the full transcript

Tool: `get_transcript` with the recording_id from Step 1.

If transcript returns empty or under 1000 words: the call was either too short or didn't transcribe well. Slack a warning and exit.

## Step 3: Identify 5 standout moments

Read the entire transcript. Score every moment that fits ONE of these five patterns:

### A. Member Question + Coach Answer
A real question from a member (or call attendee) that represents a pain point the audience has. Capture:
- The question (cleaned up to be standalone)
- The coach's key answer points (use the coach's actual words)
- Why this is a strong moment (specific, relatable, unique answer)

### B. Framework / Teaching Moment
The coach explains a process or concept with structure. Capture:
- The framework name (if mentioned) or a working title
- The steps or elements (use the coach's actual phrasing)
- Why it's quotable

### C. Story with Specific Numbers or Names
A real example with verifiable details. Capture:
- The story in 1-3 sentences
- The specific proof (number, name, before/after)
- Why it lands (relatability, specificity, surprise)

### D. Contrarian Take
The coach disagrees with conventional wisdom and backs it up. Capture:
- The conventional belief
- The coach's counter-take
- The reasoning or evidence

### E. Breakthrough Moment
A member has an "aha" moment, or the coach calls out a wrong assumption that just shifted. Capture:
- What was believed before
- What shifted
- The exact phrase that caused the shift (if there is one)

## Step 4: Quality gate (filter down to 5)

If you found more than 5 candidates, apply these filters and keep the strongest 5:

1. Would the coach's audience stop scrolling for this hook?
2. Is there a specific number, name, or example?
3. Could someone get this exact insight from any other coach? If yes, downgrade.
4. Does the coach have unique credibility to say this?

If you found fewer than 5 (call wasn't dense enough), output what you found and note the count. Don't fill with weak material.

## Step 5: Write the scripts

For each of the 5 (or N) moments, write a short-form script following the structure in `reference/short-form-tone.md`:

```
**Script [N]: [Working title]**

**Hook (5-7 sec):**
[The first line. Stops scrolling. Specific. Direct.]

**Body (20-40 sec):**
[The insight, framework, story, or contrarian take. Use the coach's actual words from the transcript where possible. Short sentences.]

**CTA (3-5 sec):**
[One specific next step.]

---

**Source quote:** "[exact text from transcript]"
**Timestamp:** [mm:ss to mm:ss if available]
**Moment type:** [A/B/C/D/E from Step 3]
```

Write each script in the coach's voice using `reference/voice-example.md` as the tone reference.

Keep total script length to 30-60 seconds spoken at normal pace (roughly 80-150 words for the full script).

## Step 6: Post scripts to Slack

Use the Slack MCP (Connectors UI handles auth). Channel: `#content-reports`.

Post a single Slack message formatted as follows. Keep the full scripts inline so the coach can read them right in Slack and copy any one out when ready to film.

```
🎬 *5 New Short-Form Scripts*
*Source:* [Call Type] - [Date]

*Script 1: [Working title]*
> Hook: [hook]
> Body: [body]
> CTA: [CTA]
_Source quote:_ "[exact text from transcript]"
_Timestamp:_ [mm:ss] | _Type:_ [A/B/C/D/E]

---

*Script 2: ...*
[same format]

---

[Repeat for all surviving scripts]
```

If only 3 or 4 scripts survived the quality gate, post that many and include a one-line note: "Generated [N] scripts from [M] candidates. Other moments filtered for being too generic."

Slack message limit is ~40k chars per post. If 5 scripts overrun the limit, split into 2 posts and add "Part 1 of 2 / Part 2 of 2" to the header.

## Step 7: Error handling

If Fathom transcript fetch fails: post `⚠️ Fathom transcript fetch failed: [error first 200 chars]` to Slack via MCP and exit.

If Slack post fails: log the error in the Routine run log (Anthropic dashboard) and exit. Don't retry. Daily/weekly cadence means next run is the safety net.

## Required env vars

None. Fathom + Slack both handled via MCP / Connectors. Just authorize them in claude.ai → Settings → Connectors before first run.

## Schedule recommendation

`0 17 * * 3,4` — Wednesday and Thursday at 5pm (after typical coaching call times). Coach can adjust to their call schedule.

For coaches with different call days, edit the cron:
- Daily coaching calls: `0 17 * * 1-5` (every weekday at 5pm)
- Monday calls only: `0 17 * * 1`
- Twice a week different days: `0 17 * * 2,4` (Tue/Thu)

## Quality rules

- Pull script content from the transcript. Don't invent insights the coach didn't say.
- If the coach said it differently than the script reads, prefer their original phrasing
- Vary the 5 scripts in format (story, take, list, contrarian, framework) so the coach has variety to film
- Skip moments where the coach was uncertain, asking a question themselves, or correcting an earlier statement
- 3 strong scripts beat 5 weak ones. The Notion page should be high signal.
