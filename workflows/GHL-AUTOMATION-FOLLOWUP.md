# GHL Follow-Up Automations — Build in GHL UI

## Automation 1: "Post-Walkthrough → Send Proposal"
**Trigger:** Opportunity moved to "Walkthrough Completed" stage
**Actions:**
1. Wait 4 hours (time to prepare proposal)
2. Send internal reminder: "Prepare proposal for {{contact.company}}"
3. Wait 24 hours
4. If opportunity NOT moved to "Proposal Sent":
   - Send escalation to Wade: "Proposal overdue for {{contact.company}}"

---

## Automation 2: "Proposal Sent → Follow Up"
**Trigger:** Opportunity moved to "Proposal Sent" stage
**Actions:**
1. Wait 48 hours
2. Send SMS:
   ```
   Hey {{contact.first_name}}, just checking in — did you get a chance to look over the proposal? Happy to answer any questions.
   ```
3. Wait 3 days
4. Send email:
   **Subject:** Following up on your cleaning proposal
   ```
   Hey {{contact.first_name}},

   Wanted to circle back on the proposal we sent over. Let me know if you have any questions or if there's anything we should adjust.

   We'd love to earn your business.

   — Alex
   Ascent Cleaning Solutions
   615-437-5660
   ```
5. Wait 4 days
6. Send final SMS:
   ```
   {{contact.first_name}} — last follow-up on the cleaning proposal. If timing isn't right, no worries at all. We're here whenever you're ready.
   ```
7. If no response after 14 days total:
   - Move to "Follow-Up" stage
   - Add tag: `proposal-no-response`
   - Create task: "Manual call — re-engage {{contact.name}}"

---

## Automation 3: "Closed Won → Onboarding"
**Trigger:** Opportunity moved to "Closed Won" stage
**Actions:**
1. Send congratulations email:
   **Subject:** Welcome to Ascent Cleaning Solutions!
   ```
   {{contact.first_name}},

   We're excited to work with you. Here's what happens next:

   1. Our operations team will reach out within 24 hours to confirm your cleaning schedule
   2. We'll finalize the scope based on our walkthrough
   3. First service within 5 business days of agreement signing

   If you need anything before then: 615-437-5660

   Looking forward to a great partnership.

   — The Ascent Cleaning Solutions Team
   ```
2. Send internal notification to Lee + Wade: "🎉 NEW CLIENT: {{contact.company}}"
3. Create task: "Schedule first cleaning for {{contact.company}}" — assigned to Lee
4. Add tags: `client`, `active-account`

---

## Automation 4: "Closed Lost → Win-Back Sequence"
**Trigger:** Opportunity moved to "Closed Lost" stage
**Actions:**
1. Add tag: `lost-deal`
2. Wait 30 days
3. Send SMS:
   ```
   Hey {{contact.first_name}}, this is Alex from Ascent Cleaning. Just checking in — has anything changed with your cleaning situation? We'd still love to earn your business.
   ```
4. Wait 60 days
5. Send email:
   **Subject:** Still here if you need us
   ```
   {{contact.first_name}},

   Just a quick note — we're still available if your cleaning needs have changed. No pressure, just wanted you to know we're here.

   — Alex
   ```
6. After 90 days: remove from sequence, add tag `win-back-complete`

---

## Automation 5: "No-Show → Reschedule"
**Trigger:** Calendar event = no-show / missed
**Actions:**
1. Send SMS immediately:
   ```
   Hey {{contact.first_name}}, looks like we missed each other today. No worries — want to reschedule? Here's the link:
   
   {{calendar_link}}
   ```
2. Wait 24 hours
3. If no reschedule:
   ```
   Just one more try — {{calendar_link}} whenever works for you. No pressure.
   ```
4. Wait 48 hours
5. If still no reschedule:
   - Move opportunity back to "New Lead"
   - Add tag: `no-show`
   - Internal notification: "No-show lead: {{contact.name}} — may need manual call"
