# Technical Documentation
## MISCALA-Customer-Service-Workflow

---

## Table of Contents
1. [Architecture Overview](#architecture-overview)
2. [Prerequisites](#prerequisites)
3. [Application Connections](#application-connections)
4. [Module Breakdown](#module-breakdown)
5. [Workflow Logic](#workflow-logic)
6. [Google Sheet Structure](#google-sheet-structure)
7. [Error Handling](#error-handling)
8. [Known Limitations](#known-limitations)

---

## Architecture Overview

This workflow is built entirely on **Make.com** and operates as an event-driven automation. The trigger is a new row detected in a Google Sheet linked to a Google Form. From that point, the scenario reads the customer's team selection, routes the data through a Switch Case and Router, and dispatches a Gmail notification to the correct team. For unclassified concerns, Make.com's built-in Simple Text Prompt module (powered by OpenAI GPT-5.2) performs AI-based routing before dispatching the email.

All submissions — regardless of route — update the originating Google Sheet row with a status value confirming whether or not the concern was successfully raised to the correct team.

**Platform:** Make.com (free tier)
**Trigger type:** Polling — Google Sheets watched every 15 minutes
**AI Module:** Simple Text Prompt (Beta) — no API key required, uses Make.com's built-in AI credits
**Export file:** `MISCALA_Workflow.blueprint.json`

---

## Prerequisites

| Requirement | Details |
|---|---|
| Make.com account | Free tier is sufficient |
| Google account | Must have access to Google Forms, Google Sheets, and Gmail |
| Google Form | Configured with the required fields (see below) |
| Google Sheet | Must be linked to the Google Form via the Responses tab |
| OpenAI API key | Not required — the Simple Text Prompt module uses Make.com's free built-in AI credits |

### Required Google Form Fields

| Field Label | Field Type |
|---|---|
| Full Name | Short answer |
| Email Address | Short answer |
| Subject | Short answer |
| Describe your concern | Paragraph |
| Which team does your concern belong to? | Multiple choice: `Sales Team` / `IT Team` / `Manufacturing Team` / `I don't know` |

---

## Application Connections

After importing the blueprint, each of the following connections must be re-authenticated using your own Google account credentials.

### Google Sheets (Trigger)
- **Module:** Watch New Rows
- **Connection:** Google OAuth 2.0
- **Sheet:** The Google Sheet linked to your Google Form (tab: `Form Responses 1` or equivalent)
- **Watched column:** The row index is used to track new entries

### Gmail (All Email Modules)
- **Connection:** Google OAuth 2.0
- **Modules affected:** Send an Email — Sales, Send an Email — IT, Send an Email — Manufacturing, Send an Email (AI-routed)
- **To field:** Update each module with the correct team L1 manager email address:
  - Sales Team: `sales@yourcompany.com`
  - IT Team: `it@yourcompany.com`
  - Manufacturing Team: `manufacturing@yourcompany.com`
- **Note:** Personal email addresses were used during testing. Replace with official team inboxes before production use.

### OpenAI — Simple Text Prompt (Beta)
- **Connection:** None required
- **Model:** GPT-5.2
- **Credits:** Consumes Make.com's built-in AI credits — no OpenAI account or API key needed
- **Prompt structure:** The concern text from the form is passed directly to the module with instructions to return only one of three values: `Sales`, `IT`, or `Manufacturing`

### Google Sheets (Log)
- **Module:** Update a Row
- **Connection:** Google OAuth 2.0
- **Sheet tab:** `Form Responses`
- **Purpose:** Updates the status column of the originating row to confirm the concern was raised to the correct team. A `NULL` status indicates the email was not sent successfully.

---

## Module Breakdown

| # | Module | App | Type | Purpose |
|---|---|---|---|---|
| 1 | Watch New Rows | Google Sheets | Trigger | Detects new form submissions and initiates the scenario |
| 2 | Switch Case | Make.com (built-in) | Logic | Reads the team selection field and maps it to a single routable variable |
| 3 | Router | Make.com (built-in) | Logic | Splits execution into 4 parallel branches based on the Switch Case output |
| 4 | Filter — Sales | Make.com (built-in) | Filter | Passes execution only when team selection equals "Sales Team" |
| 5 | Filter — IT | Make.com (built-in) | Filter | Passes execution only when team selection equals "IT Team" |
| 6 | Filter — Manufacturing | Make.com (built-in) | Filter | Passes execution only when team selection equals "Manufacturing Team" |
| 7 | Filter — I don't know | Make.com (built-in) | Filter | Passes execution only when team selection equals "I don't know" |
| 8 | Send an Email — Sales | Gmail | Action | Forwards formatted concern to Sales Team L1 manager inbox |
| 9 | Send an Email — IT | Gmail | Action | Forwards formatted concern to IT Team L1 manager inbox |
| 10 | Send an Email — Manufacturing | Gmail | Action | Forwards formatted concern to Manufacturing Team L1 manager inbox |
| 11 | Simple Text Prompt | OpenAI (via Make) | AI Action | Analyzes concern text and returns: `Sales`, `IT`, or `Manufacturing` |
| 12 | Router #2 | Make.com (built-in) | Logic | Routes AI output to the correct Gmail module |
| 13 | Send an Email (AI-routed) | Gmail | Action | Forwards concern to AI-determined team inbox with AI routing note appended |
| 14 | Update a Row | Google Sheets | Action | Updates the submission row status to confirm successful routing |

---

## Workflow Logic

```
[Google Sheets] Watch New Rows
        ↓
[Switch Case] Read team selection field
        ↓
      [Router]
  ↙     ↓      ↘        ↘
Sales   IT   Mfg    "I don't know"
  ↓     ↓     ↓           ↓
Gmail  Gmail  Gmail   [Simple Text Prompt — GPT-5.2]
                              ↓
                          [Router #2]
                        ↙    ↓     ↘
                     Sales   IT   Manufacturing
                       ↓     ↓         ↓
                     Gmail  Gmail     Gmail
                                  (AI-routed note)
          ↓ (all routes converge)
   [Google Sheets] Update a Row — Status
```

### Switch Case Logic
The Switch Case module reads the value from the team selection column of the Google Sheet and maps it to a clean variable used by the Router. This prevents routing errors caused by minor text inconsistencies from the form.

### AI Prompt Design
The Simple Text Prompt module uses the following prompt structure:

```
You are a customer service routing assistant.
Based on the customer's concern below, determine which team
should handle it. Reply with ONLY one of these three words:
Sales, IT, or Manufacturing.

Customer concern: [Concern field from Google Sheet]
```

Returning only one word ensures Router #2 can reliably match the output using simple text contains filters.

### Status Update Logic
After each route completes its Gmail send, the Update a Row module writes back to the originating row in Google Sheets. If the status column shows `NULL`, this indicates the email module did not execute — most commonly caused by a filter mismatch or a failed Gmail connection.

---

## Google Sheet Structure

### Form Responses Tab (Trigger + Log)

| Column | Source | Description |
|---|---|---|
| Timestamp | Google Forms | Auto-generated on submission |
| Full Name | Google Forms | Customer's full name |
| Email Address | Google Forms | Customer's email |
| Subject | Google Forms | Brief subject of the concern |
| Describe your concern | Google Forms | Full concern text passed to Gmail and AI |
| Which team does your concern belong to? | Google Forms | Drives the Switch Case routing logic |
| Status | Make.com (Update a Row) | Updated after routing — NULL if not sent |

---

## Error Handling

| Scenario | Behaviour |
|---|---|
| Gmail connection fails | Module throws an error; Make.com logs it in the scenario execution history |
| AI returns unexpected output | Router #2 finds no matching filter; row status remains NULL |
| Google Sheet connection fails | Trigger does not fire; scenario does not execute |
| Duplicate form submissions | Each new row triggers a separate execution — no deduplication logic is currently implemented |

To monitor errors, navigate to **Scenario → History** inside Make.com. Each execution is logged with a status of Success, Warning, or Error, along with the specific module where the failure occurred.

---

## Known Limitations

- The scenario polls every **15 minutes** on the free tier. Submissions made between polling intervals will be processed on the next scheduled run, not instantly.
- The Simple Text Prompt module consumes Make.com's built-in AI credits. Monitor credit usage under **Organization → AI Credits** in your Make.com account.
- No deduplication logic is implemented. If a row is processed twice (e.g., due to a manual re-run), a duplicate email may be sent.
- Team email addresses are placeholders and must be updated before production deployment.
