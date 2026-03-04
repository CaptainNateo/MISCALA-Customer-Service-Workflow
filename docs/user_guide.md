# User Guide
## MISCALA-Customer-Service-Workflow

---

## What Is This?

This workflow makes sure that whenever a customer submits a concern, it automatically lands in the right team's inbox — without anyone having to manually read, sort, or forward it. If the customer is unsure which team to contact, an AI will read their concern and figure it out for them.

No manual follow-up. No concerns falling through the cracks.

---

## What Does the Customer Need to Do?

The customer simply fills out a **Google Form**. It asks for:

- Their **full name**
- Their **email address**
- A **subject line** for their concern
- A **description** of their concern
- Which **team** they think can help them:
  - Sales Team
  - IT Team
  - Manufacturing Team
  - I do not know

That's it. Once they hit **Submit**, everything else happens automatically.

---

## What Happens After the Form Is Submitted?

### If the customer selected a team:
The concern is immediately sent as an email to that team's manager. The email contains the customer's name, email address, subject, and full description of their concern.

### If the customer selected "I do not know":
The workflow sends the concern to an AI model, which reads the description and determines which team is best suited to handle it. The concern is then automatically forwarded to the correct team's inbox — with a note in the email indicating that the routing was done by AI.

### In all cases:
The submission is recorded in a tracking sheet. A status is updated to confirm the concern was successfully sent to the correct team. If the status shows blank or no update, it may indicate a technical issue that needs to be checked.

---

## How Will the Team Know They Received a Concern?

Each team's L1 manager will receive an email in their inbox with the following information:

- **Customer Name**
- **Customer Email**
- **Subject**
- **Full Description of the Concern**
- *(For AI-routed emails only)* A note stating: "This concern was automatically routed by AI based on the customer's description."

No additional action is needed to receive these emails — they arrive directly in the inbox just like any other email.

---

## What Does a Successful Run Look Like?

1. Customer submits the Google Form
2. Within 15 minutes, the correct team receives an email with the concern details
3. The Google Sheet tracking log shows a status update on the customer's submission row confirming it was sent

---

## What If Something Goes Wrong?

| Situation | What It Means |
|---|---|
| The team did not receive an email | The workflow may not have run yet — it checks for new submissions every 15 minutes |
| The status column in the sheet shows no update | The concern may not have been routed successfully — flag this for the automation team |
| The wrong team received the email | The AI may have misread the concern — the team should manually forward it and notify the automation team |
| The customer submitted the form but nothing happened | Check that the Google Form is still linked to the correct Google Sheet |

For any persistent issues, contact the person managing the Make.com scenario and provide the timestamp of the affected form submission.

---

## Frequently Asked Questions

**Do customers need an account to submit a concern?**
No. The Google Form is publicly accessible. Anyone with the link can submit a concern.

**How long does it take for the team to receive the email?**
The workflow checks for new submissions every 15 minutes. In most cases, the team will receive the email within 15 minutes of the form being submitted.

**What happens if the customer selects the wrong team?**
The concern will still be sent — but to the team the customer selected. The receiving team should manually forward the concern to the correct team if needed.

**Is the customer notified after submitting?**
Google Forms displays a confirmation message after submission. However, this workflow does not currently send a follow-up email to the customer confirming receipt. This can be added as a future enhancement.

**Who manages the workflow?**
The workflow is managed through Make.com. Any changes to routing logic, team email addresses, or the form structure should be coordinated with the person responsible for maintaining the scenario.
