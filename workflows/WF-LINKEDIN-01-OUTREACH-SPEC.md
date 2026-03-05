# WF-LINKEDIN-01 — LinkedIn Outreach System Spec
# Tool: Waalaxy (Chrome Extension)
# Integration: Waalaxy → N8N webhook → GHL

## Overview
LinkedIn outreach runs through Waalaxy for automation + N8N for tracking + GHL for pipeline.

## Waalaxy Setup

### Campaign Structure (One Per ICP Segment)
Create separate Waalaxy campaigns for each segment:

1. **Campaign: Property Managers**
   Connection note: "Hi {{firstName}} — I work with property managers to support reliable, inspection-based cleaning for commercial properties. Would love to connect."

2. **Campaign: Facility Managers**
   Connection note: "Hi {{firstName}} — we help offices maintain consistent cleaning standards with dependable communication and service. Open to connecting."

3. **Campaign: Gym Owners**
   Connection note: "Hi {{firstName}} — we support fitness centers with reliable sanitation and consistent facility cleaning. Would love to connect."

4. **Campaign: Retail / Strip Centers**
   Connection note: "Hi {{firstName}} — we help retail spaces maintain a clean, welcoming environment with reliable recurring cleaning. Would love to connect."

5. **Campaign: Churches**
   Connection note: "Hi {{firstName}} — we help churches maintain clean, welcoming facilities with respectful and reliable cleaning service. Would love to connect."

6. **Campaign: Banks / Financial**
   Connection note: "Hi {{firstName}} — we support financial institutions with professional, background-checked cleaning teams. Would love to connect."

7. **Campaign: Schools / Daycares**
   Connection note: "Hi {{firstName}} — we help schools and daycares maintain safe, clean environments with structured cleaning standards. Would love to connect."

8. **Campaign: Medical / Dental**
   Connection note: "Hi {{firstName}} — we support medical offices with compliance-aware, precision cleaning protocols. Would love to connect."

### Waalaxy Sequence (Same for All Campaigns)

**Step 1:** Send connection request with segment-specific note
**Step 2:** Wait for acceptance (auto-detected by Waalaxy)
**Step 3 (Day 0-1 after accept):** Send Touch 1 message:
"Thanks for connecting, {{firstName}}. We help [facility type] maintain reliable cleaning standards with clear communication and accountability. Quick question — are you currently using an outside cleaning vendor or handling cleaning internally?"

**Step 4:** Wait 4-7 days
**Step 5:** Send Touch 2 message:
"If helpful, I can send our Facility Walkthrough Checklist. It helps identify cleaning gaps and clarify expectations for any facility. If you'd like, I can also do a quick walkthrough and provide a clear cleaning scope summary."

**Step 6:** Wait 5-7 days
**Step 7:** Send Touch 3 message:
"Totally fine if timing isn't right. Would it make sense for me to circle back next month, or is cleaning already handled long-term?"

### Waalaxy Daily Limits
- Connection requests: 20-30/day (LinkedIn safe zone)
- Messages: 50-70/day
- Profile views: 80-100/day

## N8N Integration (WF-LINKEDIN-01)

### Option A: Waalaxy Webhook → N8N
If Waalaxy supports webhooks (paid plan):
- Configure Waalaxy to send webhook on: reply received
- N8N webhook catches it → checks positive/negative → routes to GHL

### Option B: Manual Export + N8N
If no webhook:
- Export Waalaxy results CSV weekly
- N8N reads CSV → updates GHL contacts → updates data table

### N8N Workflow Structure
```
Waalaxy Webhook (or CSV import)
    ↓
Check: positive reply?
    ↓ YES                    ↓ NO
Create/update GHL contact    Update data table
Add tag: linkedin-interested  linkedin_status = not_interested
Create opportunity            
→ Walkthrough Scheduled      
Telegram alert → topic:9     
Update data table:
linkedin_status = interested
```

## Tracking (Google Sheets)
| Date | Segment | Connections Sent | Accepted | Messages Sent | Replies | Positive | Walkthroughs |
|------|---------|-----------------|----------|---------------|---------|----------|--------------|

## Success Metrics
- Connection acceptance rate: target 30-40%
- Reply rate: target 15-25%
- Positive reply rate: target 5-10%
- Walkthrough conversion: target 2-5% of connections

## Objection Handling (In Waalaxy or Manual)
- "Already have a cleaner" → "Do you review vendors periodically or is that locked into a contract?"
- "Not interested" → "No problem at all. If anything changes later this year would it be okay to check back in?"
- "Send pricing" → "Pricing depends on facility size and needs. A quick walkthrough lets us give an accurate number instead of guessing."
- "RFP only" → "Perfect. When does the next bid cycle typically open? We can do a walkthrough beforehand so everything is prepared."
