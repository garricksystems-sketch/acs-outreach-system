# ACS Outreach System

3-channel outbound outreach system for Ascent Cleaning Solutions.

## Quick Start (Tré)
1. Clone this repo
2. Open in Cursor
3. Tell Claude Code: "Read CONTEXT.md before doing anything"
4. Build workflows from there

## Files
- `CONTEXT.md` — Full system context for Claude Code (start here)
- `EMAIL_OUTREACH_SYSTEM.md` — Email strategy & architecture
- `LINKEDIN_OUTREACH_FRAMEWORK.md` — LinkedIn outreach framework
- `COLD_CALLING_FRAMEWORK.md` — Cold calling strategy (already running via ElevenLabs)
- `TRE_EMAIL_ADDON_BRIEF.md` — Build brief with GHL IDs and Claude Code prompts
- `INBOX_AGENT_SYSTEM_SPEC.md` — Inbound email triage system spec

## Workflows (N8N JSON)
- `workflows/WF-EMAIL-01-PULL-AND-QUEUE.json` — Pull leads + create GHL contacts
- `workflows/ACS_INBOX_AGENT_SYSTEM.json` — Multi-agent inbox triage

## Data Table
- ID: Z9xgFeurri2kwJ8P
- Name: leads_intake_staging
- Instance: n8n.wadeglobal.ai

## GHL Pipeline
- Pipeline: Commercial Cleaning (4BQb45bxmAFhiSCSU35e)
- Key stage: Walkthrough Scheduled (fabe9eff-921c-4a6c-a711-98e87797b3f1)
