# ACS Outreach System — Full System Map

## 3-Channel Architecture

```
                    ┌─────────────────────────┐
                    │   LEAD DATA TABLE        │
                    │   Z9xgFeurri2kwJ8P       │
                    │   leads_intake_staging    │
                    │   (enriched by WF04)      │
                    └─────┬───────┬───────┬────┘
                          │       │       │
                    Has   │  Has  │  Has  │
                    phone │ email │ LinkedIn
                          │       │       │
              ┌───────────┘       │       └───────────┐
              ↓                   ↓                   ↓
    ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
    │  CHANNEL 1      │ │  CHANNEL 2      │ │  CHANNEL 3      │
    │  AI Voice Calls │ │  Cold Email     │ │  LinkedIn       │
    │  (RUNNING)      │ │  (BUILDING)     │ │  (BUILDING)     │
    │                 │ │                 │ │                 │
    │  WF05 → EL     │ │  WF-EMAIL-01    │ │  WF-LINKEDIN-01 │
    │  Agent: Alex    │ │  → GHL sequence │ │  → Waalaxy      │
    │  Phone: 615-    │ │  reach@ inbox   │ │  → N8N webhook  │
    │  437-5660       │ │  3 emails/7days │ │  3 touches/14d  │
    └────────┬────────┘ └────────┬────────┘ └────────┬────────┘
             │                   │                   │
             ↓                   ↓                   ↓
    ┌─────────────────────────────────────────────────────────┐
    │                  INTERESTED?                             │
    │                                                         │
    │  Call: WF06 catches outcome → GHL                       │
    │  Email: WF-EMAIL-03 catches reply → GHL                 │
    │  LinkedIn: Waalaxy webhook / manual → GHL               │
    └─────────────────────┬───────────────────────────────────┘
                          ↓
    ┌─────────────────────────────────────────────────────────┐
    │              GHL PIPELINE                                │
    │  Commercial Cleaning (4BQb45bxmAFhiSCSU35e)             │
    │                                                         │
    │  New Lead → Walkthrough Scheduled → Walkthrough Done    │
    │  → Proposal Sent → Follow-Up → Closed Won/Lost         │
    └─────────────────────┬───────────────────────────────────┘
                          ↓
    ┌─────────────────────────────────────────────────────────┐
    │              INBOUND HANDLER                             │
    │  ACS Inbox Agent (Gmail)                                 │
    │                                                         │
    │  Classifier → 7 categories                              │
    │  Auto-reply: Support, Sales, Scheduling, Operations     │
    │  Route: Billing → human                                 │
    │  Silent: Confirmations, Clutter                         │
    └─────────────────────────────────────────────────────────┘
```

## File Index

| File | What It Is | Status |
|------|-----------|--------|
| `CONTEXT.md` | Full context for Claude Code | ✅ Ready |
| `SYSTEM_MAP.md` | This file — visual architecture | ✅ Ready |
| `EMAIL_OUTREACH_SYSTEM.md` | Email strategy blueprint | ✅ Ready |
| `LINKEDIN_OUTREACH_FRAMEWORK.md` | LinkedIn ICP + sequences | ✅ Ready |
| `COLD_CALLING_FRAMEWORK.md` | Calling strategy (reference) | ✅ Ready |
| `TRE_EMAIL_ADDON_BRIEF.md` | Build brief + Claude prompts | ✅ Ready |
| `INBOX_AGENT_SYSTEM_SPEC.md` | Inbox triage spec | ✅ Ready |
| `workflows/WF-EMAIL-01-PULL-AND-QUEUE.json` | N8N: pull leads + create GHL contacts | ✅ Import ready |
| `workflows/WF-EMAIL-02-GHL-SEQUENCE-SPEC.md` | GHL: 3-email sequence (build in GHL UI) | ✅ Ready |
| `workflows/WF-EMAIL-03-REPLY-DETECTION.json` | N8N: catch replies + route to pipeline | ✅ Import ready |
| `workflows/WF-LINKEDIN-01-OUTREACH-SPEC.md` | LinkedIn: Waalaxy campaigns + N8N integration | ✅ Ready |
| `workflows/ACS_INBOX_AGENT_SYSTEM.json` | N8N: Gmail inbox multi-agent triage | ✅ Import ready |

## Credentials Needed (Enter in N8N, NOT in files)
1. GHL API Key (ACS location) → for contact/opportunity creation
2. N8N API Key → for data table read/write
3. Gmail credential → for inbox agent
4. OpenRouter API Key → for inbox agent AI models
5. Telegram Bot Token → for alert notifications

## Daily Volume Targets
- AI Calls: 150-200/day (WF05)
- Cold Emails: 50/day initially → scale to 200-300 with 3 inboxes
- LinkedIn: 20-30 connections/day + 50-70 messages/day
