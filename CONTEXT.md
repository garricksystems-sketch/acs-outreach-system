# ACS OUTREACH SYSTEM — FULL CONTEXT FOR CLAUDE CODE
# Copy this entire file into CONTEXT.md in your Cursor project
# Then tell Claude Code: "Read CONTEXT.md before doing anything"

---

# COMPANY
- Name: Ascent Cleaning Solutions (ACS)
- Tagline: "Taking cleaning up a notch"
- Location: Nashville, TN
- Phone: 615-437-5660
- Services: Commercial cleaning — offices, gyms, churches, restaurants, retail, warehouses, medical, schools, daycares, dealerships, event spaces
- CRM: GoHighLevel (GHL)
- Automation: N8N (hosted at n8n.wadeglobal.ai)

---

# WHAT WE'RE BUILDING

A 3-channel outbound outreach system that books facility walkthrough appointments:

1. **AI Voice Calls** — already built and running (ElevenLabs + N8N WF05). DO NOT touch.
2. **Cold Email** — YOU are building this. GHL sequences + N8N workflows.
3. **LinkedIn Outreach** — YOU are building this. Waalaxy + N8N workflows.

Plus an **Inbox Agent** system that auto-responds to inbound emails (separate workflow, already built as JSON).

---

# N8N DATA TABLE

Table ID: Z9xgFeurri2kwJ8P
Table Name: leads_intake_staging
Instance: n8n.wadeglobal.ai

This table contains enriched leads with these columns (39 total, key ones):
- business_name
- contact_name
- phone
- email
- address / city
- status (enrichment stage)
- email_status (null = not yet emailed)
- call_status (from AI calling system)
- facility_type
- source

---

# GHL API REFERENCE (ACS Location)

Base URL: https://services.leadconnectorhq.com
API Version header: Version: 2021-07-28
Auth header: Authorization: Bearer [GHL_LOCATION_API_KEY_ACS]

## Pipeline: Commercial Cleaning
Pipeline ID: 4BQb45bxmAFhiSCSU35e

Stage IDs:
- New Lead: 1bb62f0f-8727-427f-95ab-21a8fbfac127
- Walkthrough Scheduled: fabe9eff-921c-4a6c-a711-98e87797b3f1
- Walkthrough Completed: 128bf186-6ec8-4a82-8de0-07cda9a6df0e
- Proposal Sent: 9142b7d2-63cd-4867-bb3e-940b069de777
- Follow-Up/Negotiation: 164a0b16-7f9b-41f1-9ea3-78e07fc88240
- Closed Won: 56e900cd-6edf-411e-bc12-de944d031be0
- Closed Lost: a757124e-6f63-44a6-a95c-9a9ce8f4dd2c

## GHL API Endpoints:
- Create Contact: POST /contacts/
- Update Contact: PUT /contacts/{contactId}
- Add to Workflow: POST /contacts/{contactId}/workflow/{workflowId}
- Create Opportunity: POST /opportunities/
- Update Opportunity: PUT /opportunities/{id}

---

# TAGGING ARCHITECTURE

## Contact Status Tags
Lead | Client | Active Client | Past Client | Inactive Client

## Temperature Tags
Cold Lead | Warm Lead | Hot Lead | Dead Lead

## Facility Type Tags
Medical Office | Daycare | Church | Office Building | Gym | Retail | Industrial | Warehouse | Financial Institution

## Lead Source Tags
Google Maps | Directory | Referral | Website Form | Cold Email | Cold Call | Networking | LinkedIn

## Pipeline Stage Tags
New Lead | Contacted | Replied | Walkthrough Scheduled | Proposal Sent | Won | Lost

## Service Interest Tags
General Janitorial | Floor Care | Strip and Wax | Deep Cleaning | Day Porter | Window Cleaning

## Decision Maker Tags
Property Manager | Office Manager | Owner | Facilities Director | Church Administrator

---

# EMAIL OUTREACH SYSTEM

## Architecture
Lead Sources → N8N Enrichment → GHL CRM → Email Campaigns → Replies & Pipeline Updates → Data Feedback Loop

## Email Infrastructure (HIGH VOLUME)
Multiple sending inboxes needed:
- cleaning1@[domain] → 40-60 emails/day
- cleaning2@[domain] → 40-60 emails/day  
- cleaning3@[domain] → 40-60 emails/day
Target: 200-300 emails/day total

## Email Sequence (3 emails)
Email 1 → wait 3 days → Email 2 → wait 4 days → Email 3
Rule: If reply → remove from sequence immediately

## Email Offer
10-15 minute facility walkthrough. Free, no commitment. Purpose: assess needs, identify gaps, provide pricing.

## WF-EMAIL-01: Pull & Queue Email Leads
Trigger: Schedule — daily 8AM CST
Action: 
1. Read data table Z9xgFeurri2kwJ8P
2. Filter: email != null AND email_status = null
3. Limit: 50 leads per day
4. For each: create GHL contact (firstName, lastName, companyName, email, phone)
5. Add tags: ['email-sequence-cold', 'ai-outreach', facility_type tag]
6. Enroll in GHL email workflow
7. Update data table row: email_status = 'queued'

## WF-EMAIL-02: GHL Email Sequence (build in GHL Automations)
Trigger: Contact tag added = 'email-sequence-cold'
Email 1 (Day 0): Introduction + walkthrough offer
Email 2 (Day 3): Value follow-up + checklist offer
Email 3 (Day 7): Respectful close
Stop: If reply detected
End: Add tag 'email-sequence-complete'

## WF-EMAIL-03: Reply Detection (N8N)
Trigger: GHL webhook on contact reply
Check for positive keywords: yes, interested, walkthrough, sure, sounds good
IF positive → trigger WF-EMAIL-04
IF negative → update email_status = 'not_interested'

## WF-EMAIL-04: Interested → Pipeline
Update GHL contact: remove from sequence, add tag 'email-interested'
Move to pipeline stage: Walkthrough Scheduled (fabe9eff-921c-4a6c-a711-98e87797b3f1)
Send Telegram notification
Update data table: email_status = 'interested'

---

# LINKEDIN OUTREACH SYSTEM

Tool: Waalaxy (Chrome extension)
North Star: Walkthroughs booked per week from LinkedIn

## ICP Segments (priority order)
1. Property Managers — angle: reliable vendor, inspection accountability
2. Office/Facility Managers — angle: consistency, reputation
3. Gyms/Fitness Centers — angle: high-touch sanitation
4. Retail/Strip Centers — angle: simple, reliable recurring
5. Churches — angle: respectful, event-ready
6. Banks/Financial — angle: trust, professional appearance
7. Schools/Daycares — angle: safety-focused
8. Medical/Dental — angle: compliance, precision

Rule: One segment = one message angle. Never mix.

## Connection Request Formula
Relevance → Credibility signal → Soft close

Example (Property Manager):
"Hi [Name] — I work with property managers to support reliable, inspection-based cleaning for commercial properties. Would love to connect."

## Post-Accept Sequence
Touch 1 (Day 0-1): Thank + ask if using vendor or in-house
Touch 2 (Day 4-7): Offer walkthrough checklist + walkthrough
Touch 3 (Day 10-14): Respectful close or permission to reconnect

## Three Token Rule (personalization)
Only use: 1. Role  2. Facility type  3. Desired outcome

## Objection Routing
"Already have a cleaner" → "Do you review vendors periodically?"
"Not interested" → "Would it be okay to check back in a few months?"
"Send pricing" → "A quick walkthrough lets us give an accurate number"
"RFP only" → "When does the next bid cycle open?"

---

# LEAD LIFECYCLE
Lead → Contacted → Replied → Walkthrough Scheduled → Proposal Sent → Won/Lost

---

# DATA FEEDBACK LOOP
GHL tracks: Opens, Clicks, Replies, Walkthroughs, Clients Won
N8N extracts via API → Google Sheets
Google Sheets columns: Facility Type, Subject Line, Email Style, Opens, Replies, Walkthroughs, Clients Won

---

# CORE RULES
1. GoHighLevel is the source of truth
2. N8N handles automation and data movement
3. Claude Code assists development — NOT runtime automation
4. Email campaigns focus ONLY on booking walkthroughs
5. System improves through campaign feedback data
6. DO NOT touch WF01-WF06 (existing calling pipeline)
7. Build NEW workflows only (WF-EMAIL-01 through WF-EMAIL-04)
