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

| File | What It Is | Build Where | Status |
|------|-----------|-------------|--------|
| `CONTEXT.md` | Full context for Claude Code | Reference | ✅ Ready |
| `SYSTEM_MAP.md` | This file — visual architecture | Reference | ✅ Ready |
| `EMAIL_OUTREACH_SYSTEM.md` | Email strategy blueprint | Reference | ✅ Ready |
| `LINKEDIN_OUTREACH_FRAMEWORK.md` | LinkedIn ICP + sequences | Reference | ✅ Ready |
| `COLD_CALLING_FRAMEWORK.md` | Calling strategy (reference) | Reference | ✅ Ready |
| `TRE_EMAIL_ADDON_BRIEF.md` | Build brief + Claude prompts | Reference | ✅ Ready |
| `INBOX_AGENT_SYSTEM_SPEC.md` | Inbox triage spec | Reference | ✅ Ready |
| **N8N Workflows (Import JSON)** | | | |
| `workflows/WF-EMAIL-01-PULL-AND-QUEUE.json` | Pull leads + create GHL contacts | N8N Import | ✅ Ready |
| `workflows/WF-EMAIL-03-REPLY-DETECTION.json` | Catch replies + route to pipeline | N8N Import | ✅ Ready |
| `workflows/WF-EMAIL-04-TRACKING.json` | Daily email stats → Telegram report | N8N Import | ✅ Ready |
| `workflows/ACS_INBOX_AGENT_SYSTEM.json` | Multi-agent Gmail inbox triage | N8N Import | ✅ Ready |
| **GHL Automations (Build in GHL UI)** | | | |
| `workflows/WF-EMAIL-02-GHL-SEQUENCE-SPEC.md` | 3-email cold sequence | GHL Automations | ✅ Spec Ready |
| `workflows/WF-WALKTHROUGH-BOOKING.md` | Interested → send booking link → reminders | GHL Automations | ✅ Spec Ready |
| `workflows/GHL-AUTOMATION-FOLLOWUP.md` | Post-walkthrough, proposal, win-back, no-show | GHL Automations | ✅ Spec Ready |
| **LinkedIn (Build in Waalaxy)** | | | |
| `workflows/WF-LINKEDIN-01-OUTREACH-SPEC.md` | 8 ICP campaigns + 3-touch sequences | Waalaxy | ✅ Spec Ready |

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
