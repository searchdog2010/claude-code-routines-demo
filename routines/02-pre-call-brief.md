# Routine 2: Pre-Call Sales Brief

Every weekday at 7am, the Routine queries your GHL calendar for today's sales calls. For each booked contact, it pulls their tags, intake form responses, and message history, then synthesizes a one-page brief per call. Posts to Slack so you walk into every call warm instead of cold.

---

## Step-by-step setup

### Step 1: Authorize MCP connectors (one-time)

Go to **claude.ai → Settings → Connectors** and authorize:
- **Slack** — for posting the briefs

GHL is NOT an MCP. We hit GHL's REST API directly using your Private Integration Token (env var).

### Step 2: Get your GHL Private Integration Token

(Same one used in Routine 2 if you've set up the inbox triage version. Skip if you have it.)

1. Log in to GoHighLevel
2. Click your profile icon (bottom-left) → **Settings**
3. Left sidebar → **Private Integrations**
4. Click **Create New Integration**
5. Name it: `Claude Routines`
6. Tick these scopes:
   - View Contacts
   - View Contact Custom Fields
   - View Conversations
   - View Conversation Messages
   - View Calendars
   - View Calendar Events
7. Click **Create**, copy the token (starts with `pit-...`)

### Step 3: Find your sales-call calendar IDs

You probably have one or two GHL calendars for sales calls. Find their IDs:

1. In GHL → **Settings → Calendars**
2. For each sales calendar, click into it. The URL shows the calendar ID.
3. Or hit the API: `curl https://services.leadconnectorhq.com/calendars/?locationId=<YOUR_LOC> -H "Authorization: Bearer <PIT>" -H "Version: 2021-04-15"`

For Ron's setup the two are:
- `6ogGwGObGeYiMKD5U3ur` — 30 Minute Call With Ron
- `TPgwkBAjvdu0hg0k0u0P` — 60 Minute Call With Ron

Comma-separate them in the env var.

### Step 4: Get your GHL Location ID

1. In GHL → **Settings → Business Profile**
2. The Location ID is shown there (24-character string)
3. For Ron's account: `chnYcgHTPppsu36duwll`

### Step 5: Create the Routine

1. Go to **claude.ai/code/routines**
2. Click **Create Routine**
3. Name it: `Pre-Call Sales Brief`
4. Point it at this repo
5. Paste the Routine prompt (below)
6. Add the env vars (below)
7. Set the schedule: `0 7 * * 1-5` with timezone `America/New_York`
8. Click **Save**

### Step 6: First run

1. Click **Run Now** (assuming you have at least one call booked today)
2. Wait ~2-3 minutes (longer if you have many calls)
3. Check the #sales-briefs Slack channel
4. You should see one message per booked call with the full brief
5. Read the briefs before your first call

If you have no calls today, the Routine will Slack you "no calls booked, skipping" and exit cleanly.

---

## Routine prompt (copy-paste)

```
Pre-Call Sales Brief. Be strictly directive. Run these steps in exact order. Stop the moment an exit condition fires.

STEP 1: Query GHL Calendar Events API for today's bookings.
GET https://services.leadconnectorhq.com/calendars/events
Headers: Authorization: Bearer ${GHL_API_KEY}, Version: 2021-07-28
Query params: locationId=${GHL_LOCATION_ID}, calendarId=<one of SALES_CALENDAR_IDS>, startTime=<today 00:00 in epoch ms>, endTime=<today 23:59 in epoch ms>.
Make one query per calendar in SALES_CALENDAR_IDS (comma-separated).

STEP 2: EARLY EXIT CHECK. If the combined event count across all calendars is 0:
  - Post via Slack MCP to channel #sales-briefs: "📞 No sales calls booked today (YYYY-MM-DD). Skipping brief generation."
  - STOP. Return success. Do not query anything else. Do not retry. Do not "verify" the result.

STEP 3: If the event count is >0, proceed:
For each booked contact:
  - GET https://services.leadconnectorhq.com/contacts/{contactId} (tags, custom fields, source, dateAdded)
  - GET https://services.leadconnectorhq.com/locations/{locationId}/customFields?model=contact (once, cache labels)
  - GET https://services.leadconnectorhq.com/conversations/search?locationId={loc}&contactId={cid}&limit=10
  - For each conversation: GET /conversations/{convId}/messages?limit=20

STEP 4: Classify each contact (HOT/WARM/COLD per skill rules) and build their brief following the skill's format.

STEP 5: Post ALL briefs as ONE Slack message via Slack MCP to channel #sales-briefs. Title: "📋 Sales Briefs - YYYY-MM-DD (N calls today)". One brief per call, in chronological order.

STEP 6: Done. Stop.

Hard rules:
- Read reference/voice.md and reference/niche.md ONCE at the start for context. Do not re-read them per contact.
- If any single contact fetch fails, skip that contact and continue. Do not retry.
- If the GHL calendar query returns 401, post error to Slack and stop. Do not retry.
- Do not exceed 10 API calls per contact total.
- The whole Routine should complete in under 3 minutes for typical days.
```

## Env vars

| Name | Value |
|---|---|
| `GHL_API_KEY` | Your PIT token from Step 2 (starts with `pit-`) |
| `GHL_LOCATION_ID` | Your Location ID from Step 4 |
| `SALES_CALENDAR_IDS` | Comma-separated list of sales calendar IDs from Step 3 |

Slack handled via MCP.

## Schedule

`0 7 * * 1-5` (weekdays at 7am)

Adjust if you only do calls certain days:
- Tue/Wed/Thu only: `0 7 * * 2,3,4`
- Every day including weekends: `0 7 * * *`

## What you get

A Slack message at 7am every weekday with one brief per call you have today. Each brief includes:

- Status (HOT/WARM/COLD) with emoji
- Key tags so you remember their history with you
- Their intake form responses (goal, budget, obstacle) when they filled one out
- Recent message history (filtered to remove booking-confirmation noise)
- Suggested approach tailored to their status

Read these in the 30 min before your first call. Walk in knowing more than they expect you to remember.

Cost: ~1 Routine run per weekday. Takes 2-3 minutes per run (longer if you have lots of calls).

## Quality note

This Routine is most valuable when:
- Your prospects fill out an intake form (the brief is dense with their words)
- You have GHL message history (the brief surfaces past touchpoints you'd forget)

For cold/sparse data, the brief is still useful as a "you know nothing, run a discovery call" warning. But the magic is in the intake-form-filled cases.
