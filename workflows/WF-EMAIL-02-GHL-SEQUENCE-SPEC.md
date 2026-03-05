# WF-EMAIL-02 — GHL Email Sequence (Build Inside GHL)
# This is NOT an N8N workflow — build this in GHL Automations UI

## Trigger
Contact tag added = `email-sequence-cold`

## Sequence

### Email 1 — Day 0 (Immediately)
**Subject:** Quick question about your cleaning, {{contact.first_name}}

Hey {{contact.first_name}},

I know you're busy — quick one.

We work with commercial spaces in Nashville — offices, gyms, medical, retail — and we do a free walkthrough with zero commitment.

Worth a 20-minute visit?

— Alex
Ascent Cleaning Solutions
615-437-5660

P.S. We specialize in making spaces smell great. Clients tell us that all the time.

---

### Wait 3 Days

---

### Email 2 — Day 3
**Subject:** Still open to a quick look?

Hey {{contact.first_name}} — just circling back.

If the timing wasn't right, totally get it.

We do free walkthroughs — no pitch, just an honest assessment of what your space needs. If it makes sense, we'll quote you. If not, no hard feelings.

Worth 20 minutes this week?

— Alex

---

### Wait 4 Days

---

### Email 3 — Day 7
**Subject:** Last one from me

{{contact.first_name}},

Not going to keep filling your inbox.

If you ever want a second opinion on your cleaning — or just want to see if you can get more for what you're paying — we're here.

Free walkthrough. No commitment.

ascentcleaningsolutions.com

— Alex

---

## Stop Conditions
- Contact replies → remove from workflow immediately
- Contact books appointment → remove from workflow
- Contact unsubscribes → remove + add tag `unsubscribed`

## End Action
- Add tag: `email-sequence-complete`
- Remove tag: `email-sequence-cold`

## Sender
- From: reach@ascentcleaningsolutions.com
- Name: Alex, Ascent Cleaning Solutions
- Reply-to: reach@ascentcleaningsolutions.com

## GHL Settings
- Unsubscribe link: REQUIRED in footer (CAN-SPAM)
- Track opens: YES
- Track clicks: YES
