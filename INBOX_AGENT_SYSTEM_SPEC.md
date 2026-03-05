# ACS Inbox Agent System — Architecture & Deployment Spec
**Source:** Wade's N8N workflow (March 5, 2026)
**File:** `workflows/ACS_INBOX_AGENT_SYSTEM.json`
**Status:** Ready to import — needs credential wiring
**Purpose:** AI-powered inbound email triage + auto-response for ACS Gmail inbox

---

## WHAT THIS IS

A complete multi-agent inbox system that reads every email coming into ACS Gmail, classifies it, labels it, and either:
- **Auto-responds** via an AI agent (no human needed)
- **Routes to the right human** with the email already labeled
- **Silently marks as read** for noise (confirmations, clutter)

This is **separate from the outbound email system.** This handles INBOUND. The outbound system sends cold emails. This handles what comes back — and everything else hitting the inbox.

---

## SYSTEM FLOW

```
Gmail (polls every 1 min)
         ↓
Text Classifier (GPT-4.1-mini via OpenRouter)
         ↓ (7 categories)
    ┌────┬────┬────┬────┬────┬────┬────┐
    │ AR │Sch │Fin │C/L │Int │Con │Clt │
    └─↓──┴─↓──┴─↓──┴─↓──┴─↓──┴─↓──┴─↓──┘
   Gmail Labels applied to each email
         ↓
   IF Filter (regex secondary check)
         ↓
    ┌──────────────────────────────────┐
    │ Action Required → Customer Support Agent → Reply  │
    │ Scheduling      → Scheduling Agent → Reply        │
    │ Finance/Billing → Forward to billing team (email) │
    │ Clients/Leads   → Sales & Leads Agent → Reply     │
    │ Internal Ops    → Operations Agent → Reply        │
    │ Confirmations   → Mark as read (silent)           │
    │ Clutter         → Mark as read (silent)           │
    └──────────────────────────────────┘

All AI agents share: Simple Memory (20-message window, keyed by email ID)
All AI agents use: Claude Haiku 4.5 (via OpenRouter)
```

---

## THE 4 AI AGENTS

### 1. Customer Support Agent (Action Required)
**Handles:** Complaints, missed cleans, cancellations, service failures, access issues, scope changes
**Does NOT:** Quote prices, invent contract terms, block cancellations
**Tone:** Warm, human, solution-focused
**Signs as:** `[COMPANY_NAME] Customer Support Team`

### 2. Scheduling Agent (Scheduling)
**Handles:** Reschedules, confirmations, availability questions, non-urgent schedule changes
**Does NOT:** Process cancellations, discuss pricing, handle complaints
**Tone:** Warm, brief, administrative
**Signs as:** Scheduling team implied

### 3. Sales & Leads Agent (Clients / Leads)
**Handles:** New inquiries, quote requests, walkthrough interest, proposals, referrals
**Goal:** Every response ends with a walkthrough offer — two time options
**Does NOT:** Give pricing, ask for sq footage upfront
**Signs as:** `[COMPANY_NAME] Sales Team`

### 4. Operations Agent (Internal Ops)
**Handles:** Hiring applications, internal coordination, access/key questions, staff comms
**Critical rule:** External responses ONLY — no internal notes in client-facing replies
**Signs as:** `Operations Team, [COMPANY_NAME]`

### Finance / Billing (No AI Agent)
**Handles:** Forwarded directly to `Finances.wadeglobal@gmail.com` + `JSwilliams@ascentcleaningsolutions.com`
**No AI reply** — humans handle billing

---

## MODELS USED

| Node | Model | Via |
|---|---|---|
| Text Classifier | GPT-4.1-mini | OpenRouter |
| All 4 Agents | Claude Haiku 4.5 | OpenRouter |

**⚠️ Cost note:** Both models are routed through OpenRouter (pay-per-use). Per Wade's model priority rule, Claude Haiku should ideally be called directly via Anthropic API (subscription already paid). This is a future optimization — for now OpenRouter works.

---

## DEPLOYMENT CHECKLIST — BEFORE GOING LIVE

### Step 1: Replace {{COMPANY_NAME}}
Search the workflow JSON and replace all `{{COMPANY_NAME}}` with `Ascent Cleaning Solutions` (or leave as a variable and set it in N8N credentials/env).

### Step 2: Wire Gmail Credential
- Connect the Gmail Trigger node to the ACS Gmail account (`info@ascentcleaningsolutions.com` or `reach@ascentcleaningsolutions.com`)
- Connect all Gmail action nodes (Reply, Mark as Read, Add Labels, Send) to the same credential

### Step 3: Create Gmail Labels
The workflow applies these label IDs that must exist in Gmail:
- `Label_4234360133634133691` → Action Required
- `Label_582588870732182672` → Scheduling
- `Label_131773760082095837` → Finance / Billing
- `Label_8410983963379898737` → Clients / Leads
- `Label_3048070545001841953` → Internal Operations
- `Label_7203604170059078687` → Confirmations
- `Label_4825484807859367173` → Clutter

**These label IDs are from Wade's Gmail account.** If imported into a different Gmail account, create these labels in Gmail first, then update the IDs in the workflow nodes.

### Step 4: Wire OpenRouter Credential
- Connect `GPT-4.1-mini` node → OpenRouter credential (for classifier)
- Connect `Haiku 4.5` node → OpenRouter credential (for all 4 agents)

### Step 5: Confirm Billing Forward Email
In the "Send message to billing" node — confirm recipients:
- `Finances.wadeglobal@gmail.com` ✓
- `JSwilliams@ascentcleaningsolutions.com` ✓

### Step 6: Test Before Activating
Send test emails to the connected Gmail account for each category:
1. **Action Required test:** "Our office wasn't cleaned last night and I need to know what happened"
2. **Scheduling test:** "Can we move Tuesday's cleaning to Thursday?"
3. **Billing test:** "I have a question about my last invoice"
4. **Leads test:** "Hi, I'm interested in getting a quote for our office building"
5. **Operations test:** "I'm applying for a cleaning position"
6. **Confirmations test:** Forward a system confirmation email
7. **Clutter test:** Forward a marketing/promo email

Verify: correct label applied, correct agent responds (or routes to human), response is appropriate.

### Step 7: Activate
Turn on the workflow. It will poll Gmail every minute.

---

## MEMORY ARCHITECTURE
- `Simple Memory` node provides conversation context per email thread
- Session key = Gmail message ID (each email thread has its own memory)
- Context window = 20 messages
- Shared across Customer Support, Operations, and Sales & Leads agents

---

## WHAT THIS ENABLES FOR THE OUTREACH SYSTEM

When the outbound email campaigns (WF-EMAIL-01 through WF-EMAIL-04) start generating replies, those replies hit the ACS Gmail inbox. This inbox agent system automatically:

1. Classifies the reply (likely `Clients / Leads` or `Scheduling`)
2. Fires the Sales & Leads Agent to respond with a walkthrough offer
3. Labels it so humans can review if needed

**The full loop:**
- WF-EMAIL-01 sends cold email → lead replies → Inbox Agent catches it → Sales & Leads Agent responds → books walkthrough → WF-EMAIL-04 moves to GHL pipeline

---

## WGAI PRODUCT NOTE
This inbox agent system is a **white-label WGAI product.** The `{{COMPANY_NAME}}` placeholder is intentional — swap it for any client and deploy. ACS is the proof of concept. CBN clients, Aquamen, future clients = same workflow, different credentials and name.
