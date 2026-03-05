# ACS Email Outreach Add-On — Build Brief for Tré
**Model:** Claude Sonnet 4.6 (via Cursor / Claude Code)
**Owner:** Tré
**Status:** Ready to build
**Created:** March 5, 2026

---

## WHAT YOU'RE BUILDING

An email outreach module that plugs into the existing ACS lead generation pipeline as a **parallel channel**. The AI calling system (WF01–WF06) already runs. This does NOT touch those workflows. This is a new lane — same leads, different channel.

```
EXISTING PIPELINE (DO NOT TOUCH)
WF01 Intake → WF02 Apify Leads → WF03 Dedup → WF04 Enrichment → WF05 ElevenLabs Calls → WF06 Outcomes

NEW ADD-ON (WHAT YOU'RE BUILDING)
WF04 Enrichment (already runs) → [emails land in data table]
                                        ↓
                               WF-EMAIL-01: Pull leads with emails
                                        ↓
                               WF-EMAIL-02: Cold email sequence (GHL)
                                        ↓
                               WF-EMAIL-03: Reply/open detection
                                        ↓
                               WF-EMAIL-04: Interested → GHL pipeline stage
```

---

## CREDENTIAL SITUATION — READ THIS FIRST

You are a **guest** on Wade's N8N instance (`https://n8n.wadeglobal.ai`).

**Two separate credentials:**
- **Wade's account** — owns all existing workflows (WF01–WF06), data tables, credentials
- **Your guest account** — you can build NEW workflows, but you don't have access to Wade's saved credentials

**How to handle this:**
1. Build your workflows using N8N's credential system — create NEW credentials under your guest account
2. For the GHL API key (ACS location), Wade will add it to your credential store OR you use a shared credential Wade creates
3. For the data table reads — use the N8N HTTP Request node to hit the N8N data table API (same instance, authenticated with your API key)
4. Do NOT edit WF01–WF06. Build WF-EMAIL-01 through WF-EMAIL-04 as standalone workflows

**Ask Wade before starting:**
- Can you promote Tré's guest account to allow data table read access?
- Create a shared GHL credential for ACS that Tré's workflows can use

---

## THE DATA SOURCE — N8N DATA TABLES

Wade's N8N instance has a data table called **`acs_leads`** (or similar naming) that contains enriched leads. After WF04 runs, each lead record has:

```json
{
  "id": "uuid",
  "business_name": "Ghertner Co",
  "contact_name": "John Smith",
  "phone": "615-xxx-xxxx",
  "email": "john@ghertner.com",        ← THIS IS WHAT YOU WANT
  "address": "Nashville, TN",
  "status": "enriched",                ← filter on this
  "email_status": null,                ← you'll update this
  "call_status": "no_answer",         ← from WF06
  "source": "apify_google_maps"
}
```

**Your workflow pulls leads where:**
- `email` is NOT null/empty
- `email_status` is null (not yet emailed)
- `status` = `enriched` or `ready_to_call`

---

## WORKFLOW SPECS

### WF-EMAIL-01: Pull & Queue Email Leads
**Trigger:** Schedule — runs once daily at 8:00 AM CST (before calls start)
**Purpose:** Pull leads with emails from data table, queue them for outreach

```
Schedule Trigger (8AM CST daily)
    ↓
HTTP Request → GET n8n data table (filter: has email, email_status = null)
    ↓
Limit node → max 50 leads per day (protect domain reputation)
    ↓
Loop over leads
    ↓
For each lead → create GHL contact (if not exists) + enroll in email sequence
    ↓
Update data table record → email_status = "queued"
```

**GHL Contact create/update fields:**
- firstName, lastName (split from contact_name)
- companyName = business_name
- email = email
- phone = phone
- tags: ["ai-outreach", "email-sequence", "acs-cold"]
- customField: source = "ai-pipeline"

---

### WF-EMAIL-02: Email Sequence in GHL
**This lives IN GHL, not N8N**

Build a GHL Workflow (Automation) with this sequence:

```
Trigger: Contact added to "ACS Cold Email" workflow

Day 0 (immediately):
Subject: "Quick question about your cleaning, {{contact.first_name}}"
Body:
Hey {{contact.first_name}},

I know you're busy — quick one.

We work with commercial spaces in Nashville — offices, gyms, medical, retail — 
and we do a free walkthrough with zero commitment.

Worth a 20-minute visit?

— Alex
Ascent Cleaning Solutions
615-437-5660

P.S. We specialize in making spaces smell great. Clients tell us that all the time.

---

Day 3 (if no reply, no open):
Subject: "Still open to a quick look?"
Body:
Hey {{contact.first_name}} — just circling back.

If the timing wasn't right, totally get it.

We do free walkthroughs — no pitch, just a honest assessment of what 
your space needs. If it makes sense, we'll quote you. If not, no hard feelings.

Worth 20 minutes this week?

— Alex

---

Day 7 (if no reply):
Subject: "Last one from me"
Body:
{{contact.first_name}},

Not going to keep filling your inbox.

If you ever want a second opinion on your cleaning — or just want to see
if you can get more for what you're paying — we're here.

Free walkthrough. No commitment.

ascentcleaningsolutions.com

— Alex

---

Day 7 + 1hr after email: Remove from sequence, update tag to "email-sequence-complete"
```

**GHL Sequence Settings:**
- Email sender: reach@ascentcleaningsolutions.com (Alex, Ascent Cleaning Solutions)
- Stop condition: contact replies OR books appointment
- Unsubscribe: include in footer (CAN-SPAM compliance)

---

### WF-EMAIL-03: Reply/Interest Detection (N8N)
**Trigger:** GHL webhook → fires when contact replies to email OR clicks CTA link

```
GHL Webhook (contact replied / link clicked)
    ↓
Check: is this a positive reply? (keywords: "yes", "interested", "walkthrough", "sure", "sounds good")
    ↓
IF positive → WF-EMAIL-04
IF negative/unsubscribe → update data table email_status = "not_interested", remove from GHL sequence
IF no signal → do nothing (sequence continues on schedule)
```

---

### WF-EMAIL-04: Interested → GHL Pipeline
**Trigger:** Called from WF-EMAIL-03 on positive reply

```
Input: contact_id, email content snippet
    ↓
Update GHL contact:
  - Remove from cold email sequence
  - Add tag: "email-interested"
  - Move opportunity to stage: "Walkthrough Scheduled" pipeline
    (Pipeline ID: 4BQb45bxmAFhiSCSU35e, Stage ID: fabe9eff-921c-4a6c-a711-98e87797b3f1)
    ↓
Send internal notification → Telegram topic:9 or GHL internal note:
  "🔥 Email lead interested: [name] at [company] — replied to email. Book walkthrough."
    ↓
Update N8N data table → email_status = "interested"
```

---

## GHL API REFERENCE (ACS Location)

```
Base URL: https://services.leadconnectorhq.com
Location ID: [Wade provides — stored in N8N credentials as GHL_LOCATION_ID_ACS]
API Key: [Wade provides — stored as GHL_LOCATION_API_KEY_ACS]
API Version header: Version: 2021-07-28

Create/Update Contact:
POST /contacts/

Add to Workflow (enroll in email sequence):
POST /contacts/{contactId}/workflow/{workflowId}

Create Opportunity:
POST /opportunities/

Update Opportunity Stage:
PUT /opportunities/{id}

Pipeline ID (Commercial Cleaning): 4BQb45bxmAFhiSCSU35e
Stage IDs:
  New Lead:               1bb62f0f-8727-427f-95ab-21a8fbfac127
  Walkthrough Scheduled:  fabe9eff-921c-4a6c-a711-98e87797b3f1
  Walkthrough Completed:  128bf186-6ec8-4a82-8de0-07cda9a6df0e
  Proposal Sent:          9142b7d2-63cd-4867-bb3e-940b069de777
  Closed Won:             56e900cd-6edf-411e-bc12-de944d031be0
```

---

## N8N DATA TABLE ACCESS

```
Base URL: https://n8n.wadeglobal.ai
Auth: Bearer token (your guest API key — generate in N8N Settings > API)

Read data table rows:
GET /api/v1/data-tables/{tableId}/rows
Headers:
  Authorization: Bearer {YOUR_N8N_API_KEY}
  Content-Type: application/json

Filter example (leads with email, not yet emailed):
GET /api/v1/data-tables/{tableId}/rows?filters[email][$notNull]=true&filters[email_status][$null]=true

Update a row:
PATCH /api/v1/data-tables/{tableId}/rows/{rowId}
Body: { "email_status": "queued" }
```

**Note:** Wade needs to give you the data table ID for acs_leads. Ask him before starting.

---

## WHAT TO ASK WADE BEFORE BUILDING

1. **Data table name/ID** — what is the acs_leads table ID in N8N?
2. **Credential promotion** — can Tré's guest account read data tables?
3. **GHL shared credential** — create one Tré can use for ACS API calls
4. **Email domain status** — is reach@ascentcleaningsolutions.com warmed up yet? (If not, warm-up must complete before any sequences send)
5. **GHL Workflow ID** — once you build the email sequence in GHL, get the workflow ID for the enrollment API call

---

## PROMPT FOR CLAUDE CODE (CURSOR)

Copy this exactly into Claude Code when starting:

---

**"I'm building an N8N email outreach add-on for an existing lead generation pipeline. Here's the context:**

**Existing system:** N8N instance at n8n.wadeglobal.ai runs a 6-workflow lead gen pipeline (WF01-WF06) for a commercial cleaning company (Ascent Cleaning Solutions). WF04 enriches leads and stores them in a data table with fields including: business_name, contact_name, phone, email, status, email_status, call_status.**

**What I'm building:** 4 new N8N workflows (WF-EMAIL-01 through WF-EMAIL-04) that:
1. Pull enriched leads with emails from the data table (daily at 8AM)
2. Create GHL (GoHighLevel) contacts and enroll them in a cold email sequence
3. Detect positive replies via GHL webhook
4. Move interested contacts to the GHL pipeline Walkthrough Scheduled stage and send a Telegram notification**

**Constraints:**
- I'm a guest user on the N8N instance — I can create new workflows but not edit existing ones
- GHL API version: 2021-07-28, base URL: services.leadconnectorhq.com
- Email sender: reach@ascentcleaningsolutions.com (configured in GHL)
- Must update email_status field in data table for each action (queued → sent → interested/not_interested)
- Max 50 emails per day to protect domain reputation**

**Start by helping me build WF-EMAIL-01 as an N8N JSON workflow export I can import directly."**

---

## BUILD ORDER

1. Build WF-EMAIL-01 first — get data table reads working
2. Build the GHL email sequence manually in GHL UI (easier than API)  
3. Build WF-EMAIL-03 webhook handler
4. Build WF-EMAIL-04 pipeline updater
5. Test with 3-5 leads before opening the floodgates

---

## SUCCESS LOOKS LIKE

- Lead has email in data table
- 8AM: WF-EMAIL-01 pulls them, creates GHL contact, enrolls in sequence
- Day 0: First email sends from reach@ascentcleaningsolutions.com
- Lead replies "yes I'm interested"
- WF-EMAIL-03 catches it → WF-EMAIL-04 moves them to Walkthrough Scheduled in GHL
- Telegram fires: "🔥 Email lead interested: [name]"
- Wade opens GHL and sees a real walkthrough to book

That's the win.
