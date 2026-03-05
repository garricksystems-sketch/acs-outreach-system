# ACS Outreach System

3-channel outbound outreach system for Ascent Cleaning Solutions (Nashville, TN).

## Quick Start
1. Clone this repo
2. Open in Cursor
3. Tell Claude Code: "Read CONTEXT.md before doing anything"
4. Start building

## System Map
See `SYSTEM_MAP.md` for the full visual architecture.

## Channels
| Channel | Status | Tool | Workflow |
|---------|--------|------|----------|
| AI Voice Calls | ✅ Running | ElevenLabs + N8N WF05 | DO NOT TOUCH |
| Cold Email | 🔨 Building | GHL + N8N | WF-EMAIL-01 through 03 |
| LinkedIn | 🔨 Building | Waalaxy + N8N | WF-LINKEDIN-01 |
| Inbox Agent | 📦 Ready to import | Gmail + N8N | ACS_INBOX_AGENT_SYSTEM |

## Files
- `CONTEXT.md` — Full context for Claude Code (START HERE)
- `SYSTEM_MAP.md` — Visual architecture + file index
- `EMAIL_OUTREACH_SYSTEM.md` — Email strategy blueprint
- `LINKEDIN_OUTREACH_FRAMEWORK.md` — LinkedIn ICP segments + sequences
- `COLD_CALLING_FRAMEWORK.md` — AI calling strategy (reference only)
- `TRE_EMAIL_ADDON_BRIEF.md` — Build brief with GHL IDs + Claude Code prompts
- `INBOX_AGENT_SYSTEM_SPEC.md` — Inbound email triage system

## N8N Workflows (importable JSON)
- `workflows/WF-EMAIL-01-PULL-AND-QUEUE.json` — Pull leads, create GHL contacts
- `workflows/WF-EMAIL-02-GHL-SEQUENCE-SPEC.md` — 3-email sequence (build in GHL UI)
- `workflows/WF-EMAIL-03-REPLY-DETECTION.json` — Catch replies, route to pipeline
- `workflows/WF-LINKEDIN-01-OUTREACH-SPEC.md` — Waalaxy campaign specs
- `workflows/ACS_INBOX_AGENT_SYSTEM.json` — Multi-agent Gmail inbox triage

## Data
- Table ID: Z9xgFeurri2kwJ8P (leads_intake_staging)
- Pipeline: Commercial Cleaning (4BQb45bxmAFhiSCSU35e)
- Key stage: Walkthrough Scheduled (fabe9eff-921c-4a6c-a711-98e87797b3f1)
