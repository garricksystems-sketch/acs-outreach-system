# ACS Outbound Email System — Planning Blueprint
**Source:** Wade's GPT planning session (March 5, 2026)
**Status:** Planning — Ready for build
**Owner:** Tré
**Primary CRM:** GoHighLevel (source of truth)
**Automation Engine:** N8N
**Workflow Builder:** Claude + Cursor
**Learning Layer:** Notion + Google Sheets
**Reply Handling:** VA via GHL Conversations

---

## SYSTEM GOAL
Generate commercial cleaning walkthrough appointments through high-volume outbound email.

**Funnel:** Lead Discovery → Email Outreach → Reply/Interest → Walkthrough Scheduled → Proposal Sent → Client Won

**Email's only job: BOOK WALKTHROUGHS.**

---

## CORE ARCHITECTURE

```
Lead Sources → N8N Enrichment → GHL CRM → Email Campaigns → Replies & Pipeline Updates → Data Feedback Loop → Learning Dataset
```

---

## TECHNOLOGY STACK

| Layer | Tool | Purpose |
|---|---|---|
| CRM (source of truth) | GoHighLevel | Contacts, pipelines, tags, campaigns, conversations |
| Automation | N8N | Scraping, enrichment, email validation, lead scoring, data routing |
| Dev Assistants | Claude + Cursor | Generate N8N workflow JSON, automation logic, email templates, campaign analysis |
| Learning Layer | Google Sheets | Raw campaign performance data |
| Strategic Docs | Notion | Winning messaging patterns, campaign documentation |

**Note:** Claude/Cursor are NOT runtime automation — they assist development only.

---

## LEAD DISCOVERY SOURCES
- Google Maps scraping (Apify)
- Business directories
- LinkedIn company search
- Chamber of commerce listings
- Manual VA research

---

## MINIMUM LEAD DATA FIELDS
- Company Name
- Facility Type
- City
- Phone
- Website
- Email
- Decision Maker (if known)
- Lead Source
- Lead Status

---

## EMAIL INFRASTRUCTURE — HIGH VOLUME STRATEGY

**Multiple sending inboxes required:**
```
cleaning1@[domain]   → 40–60 emails/day
cleaning2@[domain]   → 40–60 emails/day
cleaning3@[domain]   → 40–60 emails/day
```
**Target system capacity: 200–300 emails/day**

Each inbox must be warmed up independently before use.

---

## EMAIL SEQUENCE (GHL)

```
Email 1
  ↓ wait 3 days
Email 2
  ↓ wait 4 days
Email 3

Rule: If reply occurs → REMOVE from sequence immediately.
```

**Offer in every email:** 10–15 minute facility walkthrough.
**Walkthrough purpose:**
- Assess facility needs
- Identify missed cleaning areas
- Provide directional pricing range
- Explain service structure

---

## WALKTHROUGH FOLLOW-UP FLOW
After walkthrough → client receives proposal email containing:
- Thank you message
- Summary of visit
- Proposal attachment or link
- Next step instructions

Lead status → `Proposal Sent`

---

## GHL TAGGING ARCHITECTURE

**Contact Status Tags:**
`Lead` | `Client` | `Active Client` | `Past Client` | `Inactive Client`

**Temperature Tags:**
`Cold Lead` | `Warm Lead` | `Hot Lead` | `Dead Lead`

**Facility Type Tags:**
`Medical Office` | `Daycare` | `Church` | `Office Building` | `Gym` | `Retail` | `Industrial` | `Warehouse` | `Financial Institution`

**Lead Source Tags:**
`Google Maps` | `Directory` | `Referral` | `Website Form` | `Cold Email` | `Cold Call` | `Networking`

**Pipeline Stage Tags:**
`New Lead` | `Contacted` | `Replied` | `Walkthrough Scheduled` | `Proposal Sent` | `Proposal Pending` | `Won` | `Lost`

**Service Interest Tags:**
`General Janitorial` | `Floor Care` | `Strip and Wax` | `Deep Cleaning` | `Day Porter` | `Window Cleaning`

**Decision Maker Tags (optional):**
`Property Manager` | `Office Manager` | `Owner` | `Facilities Director` | `Church Administrator`

---

## DATA FEEDBACK LOOP

GHL tracks → N8N extracts via API → stored in Google Sheets

**Google Sheets Learning Dataset columns:**
- Facility Type
- Subject Line
- Email Style
- Opens
- Replies
- Walkthroughs
- Clients Won

**LLM analysis method:**
> "Analyze the outreach results dataset and generate subject line variations based on the highest reply rate patterns."

---

## CONTENT SYSTEM INTEGRATION (OPTIONAL)
Client interactions → content inputs:
- Testimonials → CIS (Content Intelligence System)
- Case studies → CES (Content Execution System)
- Cleaning insights → COS (Content Operations System)

---

## CORE OPERATING PRINCIPLES
1. GoHighLevel is the source of truth.
2. N8N handles automation and data movement.
3. LLMs assist development and analysis — not runtime.
4. Email campaigns focus on booking walkthroughs only.
5. System improves through campaign feedback data.
