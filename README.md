# MISCALA-Customer-Service-Workflow

## Overview
Customer Service has always been a real pain-point in every successful business, not to mention the routing of said customers to the correct teams. This workflow automates that entire process — from the moment a customer submits their concern, to the moment the right team receives it in their inbox — eliminating manual triage, reducing response delays, and ensuring no concern gets lost or misrouted.

## Tools Used
- **Google Forms / Google Sheets** — Customer input trigger. Whenever a customer submits the form, a new row is added to the linked Google Sheet, which activates the workflow.
- **Make.com** — Core automation orchestrator. Handles all routing logic, conditional branching, and module sequencing.
- **OpenAI (GPT-5.2)** — Used to analyze and identify the correct team who will resolve the customer's concern when the customer is unsure where to direct it.
- **Gmail** — Sends the formatted customer concern to the L1 managers of the respective team.
- **Google Sheets** — Structured logging of all submissions, including who routed the concern and which team received it.

## How It Works
The workflow begins the moment a customer completes and submits the Google Form. The linked Google Sheet detects the new row and triggers the Make.com scenario. From there, the workflow reads the customer's team selection and routes accordingly through one of four paths:

**Case 1 — Sales Team**
The customer selects "Sales Team." The workflow immediately compiles their submission details and sends a formatted concern email directly to the Sales Team's L1 manager inbox via Gmail.

**Case 2 — IT Team**
The customer selects "IT Team." The workflow follows the same process and forwards the concern directly to the IT Team's L1 manager inbox.

**Case 3 — Manufacturing Team**
The customer selects "Manufacturing Team." The concern is forwarded directly to the Manufacturing Team's L1 manager inbox without any additional processing.

**Case 4 — I Don't Know**
The customer selects "I don't know." Rather than leaving the concern unrouted, the workflow sends the concern text to OpenAI (GPT-5.2) for analysis. The model reads the concern and returns the most appropriate team — Sales, IT, or Manufacturing. The workflow then routes the email to the correct team's inbox, with a note included in the email indicating that the routing was determined by AI.

Regardless of which path is taken, every submission's status is then updated in the Google sheet, making sure each submission will be raised to the correct team. IF the submission status shows NULL, then there was no email sent to the correct channel.

## Installation / Setup Instructions

### Prerequisites
Before importing the workflow, make sure you have the following ready:
- A **Make.com** account (free tier is sufficient)
- A **Google account** with access to Google Forms, Google Sheets, and Gmail
- An **OpenAI API key** — This is no longer needed as we are using our FREE tokens within Make.com for this module
- A Google Form set up with the fields listed below, linked to a Google Sheet via the Responses tab

### Google Form Fields
| Field | Type |
|---|---|
| Full Name | Short answer |
| Email Address | Short answer |
| Subject | Short answer |
| Describe your concern | Paragraph |
| Which team does your concern belong to? | Multiple choice: Sales Team / IT Team / Manufacturing Team / I don't know |

### Importing the Blueprint
1. Log in to Make.com and go to **Scenarios**
2. Click **Create a new scenario**
3. Click the **three dots (...)** at the bottom left of the canvas
4. Select **Import Blueprint**
5. Upload `/workflow/MISCALA_Workflow.blueprint.json` from this repository
6. The full scenario will load with all modules pre-configured

### Connecting Your Accounts
After importing, reconnect each module to your own credentials:
1. **Google Sheets (trigger)** — Connect your Google account and select your linked Form Responses sheet
2. **OpenAI** — Enter your OpenAI API key
3. **Gmail modules** — Connect your Google account and update the **To** field in each module with the correct team email addresses:
   - Sales Team: `sales@yourcompany.com` || For testing purposes used personal email
   - IT Team: `it@yourcompany.com` || For testing purposes used personal email
   - Manufacturing Team: `manufacturing@yourcompany.com` || For testing purposes used personal email
4. **Google Sheets (log)** — Connect your Google account and select the sheet tab named **Form Responses**

### Activating the Scenario
1. Click **Run Once** to test with a live form submission
2. Confirm all four routes execute correctly (see `/screenshots/` for expected results)
3. Toggle the scenario **ON** using the switch at the bottom left
4. Set the scheduling interval to **every 15 minutes**

## Workflow Components

| Module | App | Purpose |
|---|---|---|
| Watch New Rows | Google Sheets | Triggers the scenario whenever a new form response row is detected |
| Switch Case | Make.com (built-in) | Helps split the choice of the user into one variable |
| Router | Make.com (built-in) | Splits the flow into 4 branches based on the customer's team selection |
| Filter — Sales | Make.com (built-in) | Allows execution only when the selected team is "Sales Team" |
| Filter — IT | Make.com (built-in) | Allows execution only when the selected team is "IT Team" |
| Filter — Manufacturing | Make.com (built-in) | Allows execution only when the selected team is "Manufacturing Team" |
| Filter — I don't know | Make.com (built-in) | Allows execution only when the customer selects "I don't know" |
| Send an Email — Sales | Gmail | Forwards the formatted concern to the Sales Team L1 manager |
| Send an Email — IT | Gmail | Forwards the formatted concern to the IT Team L1 manager |
| Send an Email — Manufacturing | Gmail | Forwards the formatted concern to the Manufacturing Team L1 manager |
| Create a Completion | OpenAI | Analyzes the concern text and returns one of three values: Sales, IT, or Manufacturing |
| Router #2 | Make.com (built-in) | Routes the AI's response to the correct Gmail module |
| Send an Email (AI-routed) | Gmail | Forwards the concern to the AI-determined team, with a note that routing was done by AI |
| Update a Row | Google Sheets | Logs every Status of each submission on whether or not their concern has been raised to the correct team |
