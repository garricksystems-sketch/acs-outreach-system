# Walkthrough Auto-Booking — GHL Automation Spec

## The Problem
When a lead says "yes" on a call/email/LinkedIn — NOTHING happens automatically.
No appointment created, no calendar link sent, no reminder. Manual gap = lost leads.

## The Fix — Build This in GHL Automations

### Trigger 1: Call Outcome = Interested
- When: Contact tag `interested` added (from WF06 callback)
- Action: Send booking link + create opportunity

### Trigger 2: Email Reply = Positive
- When: Contact tag `email-interested` added (from WF-EMAIL-03)
- Action: Send booking link + create opportunity

### Trigger 3: LinkedIn = Positive (manual tag for now)
- When: Contact tag `linkedin-interested` added
- Action: Send booking link + create opportunity

---

## GHL Automation Flow (Same for All 3 Triggers)

### Step 1: Create/Update Opportunity
- Pipeline: Commercial Cleaning (`4BQb45bxmAFhiSCSU35e`)
- Stage: Walkthrough Scheduled (`fabe9eff-921c-4a6c-a711-98e87797b3f1`)
- Source: tag-based (call / cold-email / linkedin)

### Step 2: Send Booking SMS
```
Hey {{contact.first_name}}, this is Alex from Ascent Cleaning Solutions. 

Thanks for being open to a walkthrough! Here's my calendar to pick a time that works for you:

{{calendar_link}}

It takes about 20 minutes — completely free, no commitment. Looking forward to it.
```

### Step 3: Send Booking Email
**Subject:** Your free walkthrough — pick a time

```
Hey {{contact.first_name}},

Great connecting with you. Here's the link to schedule your free walkthrough:

{{calendar_link}}

Takes about 20 minutes. We'll walk the space, note what needs attention, and follow up with a clear scope and quote.

No commitment. No pressure.

— Alex
Ascent Cleaning Solutions
615-437-5660
```

### Step 4: Wait 24 hours

### Step 5: If NO appointment booked → Send reminder SMS
```
Hey {{contact.first_name}}, just circling back — here's the walkthrough link again whenever you're ready:

{{calendar_link}}

No rush. We're here when you are.
```

### Step 6: Wait 48 hours

### Step 7: If STILL no appointment → Internal notification
- Notify Wade/Lee via Telegram topic:9
- "Lead {{contact.name}} / {{contact.company}} hasn't booked after 3 days. Manual follow-up needed."

---

## Calendar Settings in GHL
- Calendar: ACS Walkthroughs (`$GHL_CALENDAR_ID_ACS_WALKTHROUGHS`)
- Duration: 30 minutes (20-min walkthrough + 10 buffer)
- Availability: Mon-Fri 8AM-5PM CST
- Buffer between appointments: 30 minutes (travel time)
- Confirmation email: ON
- Reminder SMS: 2 hours before
- Reminder email: 24 hours before

## After Walkthrough Completed
- Move opportunity to: Walkthrough Completed (`128bf186-6ec8-4a82-8de0-07cda9a6df0e`)
- Trigger: Calendar event completed (or manual tag)
- Next: Generate and send proposal → Proposal Sent stage

---

## Build Steps (In GHL Automations UI)

1. Create automation → name: "ACS — Interested Lead → Book Walkthrough"
2. Trigger: Contact Tag Added = `interested` OR `email-interested` OR `linkedin-interested`
3. Add action: Create/Update Opportunity (pipeline + stage above)
4. Add action: Send SMS (booking template above)
5. Add action: Send Email (booking template above)
6. Add wait: 24 hours
7. Add condition: If appointment NOT booked
8. Add action: Send SMS (reminder template)
9. Add wait: 48 hours
10. Add condition: If appointment STILL not booked
11. Add action: Send internal notification (Telegram or GHL notification)
