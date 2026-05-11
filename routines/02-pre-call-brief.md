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
Run the pre-call-brief skill in this repo.

Read reference/voice.md and reference/niche.md for context first.

Use the GHL Calendar Events API to find today's bookings on these calendars (comma-separated in SALES_CALENDAR_IDS env var). Authentication via GHL_API_KEY (Bearer) and GHL_LOCATION_ID headers. Version header: 2021-07-28.

For each booked contact today:
1. Pull their full GHL contact profile (tags, custom fields, source, date added)
2. Pull the custom field schema once (caches labels for the run)
3. Pull their conversation history (last 10 conversations, 20 messages each)
4. Classify status as HOT (paid member), WARM (lead with intake or in nurture), or COLD (brand new)
5. Build a one-page brief with: status, key tags, intake form highlights (prioritize strategic fields like budget, goal, revenue), recent notable messages (skip booking-confirmation boilerplate), and a tailored suggested approach
6. Post all briefs to Slack as one message via Slack MCP, channel #sales-briefs

If zero calls today: Slack "No sales calls booked today" and exit.

If a contact fetch fails: skip that contact, continue with the others, note the skip count.

Use the priority fields and approach heuristics defined in the skill. Don't fabricate intake answers if they don't exist. For sparse-data contacts, the brief should still be useful by recommending full discovery.

Quality bar: surface what the coach would have forgotten. Tags, the specific obstacle from the intake form, the date of last contact, any "no show" flags. The brief is most valuable when it catches the thing you'd have missed.
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
