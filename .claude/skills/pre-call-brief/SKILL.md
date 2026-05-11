---
name: pre-call-brief
description: "Generate a one-page sales brief for today's GHL calendar bookings. Pulls each booked contact's tags, intake form responses, and conversation history. Synthesizes who they are, what they want, what to lead with, and likely objections. Posts to Slack. Runs as a Claude Code Routine, daily morning."
---

> **ABSOLUTE RULE: NO DASHES FOR PUNCTUATION.** Never use em dashes, en dashes, or double hyphens (--) as punctuation in output. Use commas, periods, or rewrite.

# Pre-Call Sales Brief

Runs every weekday morning. Looks at today's sales calls on the GHL calendar. For each booked contact, pulls everything we know about them and writes a one-page brief. Coach reads it before the call and walks in warm instead of cold.

## Step 0: Load Context

Read these reference files first:
- `reference/voice.md` — coach's tone (used for the "what to lead with" suggestions)
- `reference/niche.md` — coach's audience and offer (used for classification heuristics)

## Step 1: List today's bookings

Query the GHL Calendar Events API. Endpoint: `GET https://services.leadconnectorhq.com/calendars/events`

Headers:
- `Authorization: Bearer ${GHL_API_KEY}`
- `Version: 2021-07-28`

Query params:
- `locationId=${GHL_LOCATION_ID}`
- `calendarId=` for each calendar that's a sales call (configure in the Routine prompt)
- `startTime=` today at 00:00 (epoch ms)
- `endTime=` today at 23:59 (epoch ms)

Default calendars to scan (Ron's setup, adjust for other coaches):
- `6ogGwGObGeYiMKD5U3ur` — 30 Minute Call With Ron
- `TPgwkBAjvdu0hg0k0u0P` — 60 Minute Call With Ron

For each appointment, capture: contact ID, title, startTime.

If zero appointments today: Slack `📞 No sales calls booked today. Skipping brief generation.` and exit cleanly.

## Step 2: For each contact, pull everything

For each contact ID from Step 1, hit three GHL endpoints in parallel:

### 2a. Contact profile

`GET https://services.leadconnectorhq.com/contacts/{contactId}`
Headers: Authorization Bearer + Version 2021-07-28

Capture: firstName, lastName, email, phone, source, dateAdded, tags, customFields (array of {id, value}).

### 2b. Custom field schema (once, cached)

`GET https://services.leadconnectorhq.com/locations/{locationId}/customFields?model=contact`
Headers: same.

This maps custom field IDs to human-readable labels. Build a dict once at the start of the run.

### 2c. Conversation history

`GET https://services.leadconnectorhq.com/conversations/search?locationId={locationId}&contactId={contactId}&limit=10`

For each conversation returned, hit:
`GET https://services.leadconnectorhq.com/conversations/{conversationId}/messages?limit=20`

Capture the last 15-20 messages per conversation. Filter to non-confirmation messages (drop "Thanks for scheduling", "reminder for our call", "starts in 10 minutes" boilerplate).

## Step 3: Classify each contact's status

Based on tags + intake fields + conversation activity:

- 🔥 **HOT - Paid member**: Tags include "paid community - member" or similar. Likely call is about upgrade or coaching, not new sale.
- 🌡️ **WARM - Lead with intake history**: Filled out intake form, has tags showing engagement (webinar promo, course interest, etc.). Discovery already partially done.
- 🌡️ **WARM - In nurture flow**: Has tags showing email/webinar campaign engagement but no intake form. Still in discovery mode.
- ❄️ **COLD - Brand new**: No tags, no intake, no message history beyond booking confirmation. Full discovery required.

## Step 4: Build the brief

For each contact, produce a Slack-formatted brief:

```
*[Name]*  |  [Day Time, e.g. "Tue 11:00am"]

*Status:* [emoji + status line]
*Email:* [email]
*First touch:* [source] on [YYYY-MM-DD]
*Tags:* [comma-separated tags or "none"]

*What they want* (from intake form):
- *[Field label]:* [value, max 200 chars]
- *[Field label]:* [value]
(only include the strategic fields, see Priority Fields below)

*Conversation history:* [N] messages total ([N inbound] / [N outbound])

*Recent notable messages:*
- [date] ([direction]): [first 180 chars, skip boilerplate]
- [date] ([direction]): [...]

*Suggested approach:*
- [Tailored to status; see Approach Heuristics below]
- [2-4 bullets max]
```

### Priority intake fields (rank these first)

Surface these labels when present, in this order:
1. "What best describes your current situation"
2. "What's happening in your life right now"
3. "How serious are you"
4. "What do you sell" / "current monthly revenue" / "desired monthly revenue"
5. "stopping you" (their stated obstacle)
6. "Investment Budget" / "Financial Resources"
7. "Date - Last Sales Call" (when they last spoke to a sales rep)
8. "Invoice Amount Paid" (if they've paid before)

Skip generic system fields (Skool ID, system UUIDs, raw timestamps without context).

### Approach Heuristics

**For 🔥 HOT (paid member):**
- Existing paid member, so this is likely an upgrade or coaching question, not a new sale
- Lead with: "What's working / not working since you joined?" before pitching anything
- Watch for upgrade signals (asking about higher-touch tiers, specific gaps in current tier)
- If tags contain "no show": ⚠️ flag flakiness, confirm timing on the call open

**For 🌡️ WARM with intake form filled:**
- Intake form was filled out, so lean on those specifics. Quote their own goal back to them.
- Skip discovery on basics, dig into the specific obstacle they named in the form

**For 🌡️ WARM in nurture but no intake:**
- In nurture but never filled intake, so still discovery mode
- Reference the campaign they're in (e.g., webinar promo) to start the conversation

**For ❄️ COLD:**
- Cold lead, brand new. Full discovery call.
- Open with: "What made you book this call?" and let them lead
- Standard offer overview only if they ask. Don't pitch unprompted.

## Step 5: Post all briefs to Slack

Use the Slack MCP. Channel: `#sales-briefs` (or whatever channel the coach prefers).

Post as ONE message titled `📋 Sales Briefs - [Date] (N calls today)`. List each brief with a horizontal rule between them.

If 5+ calls, split into multiple posts (Slack has a 40k char per-message cap). Header each: "Part 1 of N" etc.

## Step 6: Error handling

- If GHL API returns 401: Slack `⚠️ Pre-Call Brief failed: GHL auth invalid. Check GHL_API_KEY.` Exit.
- If a single contact fetch fails: skip that contact, continue with the rest. Note the skip count in the final message.
- If Slack post fails: log in Routine dashboard, no retry. Daily cadence is the safety net.

## Required env vars

- `GHL_API_KEY` — GoHighLevel Private Integration Token
- `GHL_LOCATION_ID` — GHL sub-account location ID
- `SALES_CALENDAR_IDS` — comma-separated list of GHL calendar IDs to scan (or hardcode in prompt)

Slack handled via MCP / Connector.

## Schedule recommendation

`0 7 * * 1-5` — every weekday at 7am

The brief should arrive in Slack before the coach's first call of the day.

## Quality rules

- Don't fabricate. If intake fields aren't there, say so plainly. ("No intake form filled.")
- Surface contradictions when they exist (e.g., paid member with "no show" tag, or warm lead who hasn't replied to any nurture emails)
- Match the coach's voice in the "suggested approach" lines (mentor energy, not formal)
- For sparse data cases, the brief should still be useful as a "you know nothing about this person, here's how to run a real discovery call"
- No em dashes, en dashes, or double hyphens
