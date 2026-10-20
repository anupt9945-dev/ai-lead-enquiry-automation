# AI Lead & Customer Enquiry Assistant Automation

An n8n workflow that automatically handles new business enquiries. When a lead fills a Google Form, the workflow reads the enquiry with AI, scores it, sends a personalised reply to the lead, and alerts the business owner by email. All of it happens within about a minute, with no manual work.

Built by **Anup Tiwari** (AI Agent & Automation Workflows).

---

## What it does

1. A lead submits a **Google Form**, and the answer is saved as a new row in Google Sheets.
2. **n8n** detects the new row automatically.
3. **Google Gemini** reads the message and works out:
   - lead **score** (for example Hot or Warm)
   - **intent** (what the lead actually wants)
   - **urgency**
   - a ready-to-send, personalised **reply**
4. The reply is **emailed to the lead** through Gmail.
5. A **lead alert** with the score and details is emailed to the business owner.
6. If anything fails, a separate **Error Alert workflow** emails the failure details and an execution link.

## Workflow overview

```
Google Form
    |
Google Sheets (new row)
    |
Google Sheets Trigger  ->  Edit Fields  ->  IF
                                             |
                       Basic LLM Chain (Gemini Flash)
                                             |
                          Code (split the AI JSON output)
                                             |
                      +----------------------+----------------------+
                      |                                             |
        Gmail: reply to the lead                    Gmail: lead alert to owner

Any failure  ->  Error Alert workflow (Error Trigger -> Gmail)
```

## Nodes used

**Main workflow**

| # | Node | Purpose |
|---|------|---------|
| 1 | Google Sheets Trigger | Watches the Leads sheet for new rows (checks every minute) |
| 2 | Edit Fields | Sets business name, sender name and business description used in the AI prompt |
| 3 | IF | Checks that the lead's email looks valid (contains @) before the AI step |
| 4 | Basic LLM Chain | Sends the lead message to the AI with a structured prompt |
| 5 | Google Gemini Chat Model | The AI model used by the chain (Gemini Flash, free tier) |
| 6 | Code | Splits the AI JSON into clean fields (score, intent, urgency, reply and so on) |
| 7 | Gmail | Sends the personalised reply to the lead |
| 8 | Gmail | Sends the lead alert to the business owner |

**Error Alert workflow (separate)**

| # | Node | Purpose |
|---|------|---------|
| 1 | Error Trigger | Starts automatically when the main workflow fails |
| 2 | Gmail | Emails the workflow name, failed node, error message and execution link |

## Reliability

- **Retry On Fail:** the AI node (Basic LLM Chain) retries automatically up to **3 times** with a **3-second wait** between tries. This handles short Gemini outages such as "service unavailable" without any action from the owner.
- **Error alerts:** if all retries fail, the Error Alert workflow emails the problem, so no lead is silently missed.
- **Free-tier note:** the free Gemini API has a per-minute request limit. Normal lead flow stays well within it. For bulk imports, add a Loop Over Items and Wait node to process rows one by one.

## Tech stack

- n8n Cloud
- Google Sheets and Google Forms
- Google Gemini (AI Studio API)
- Gmail

## Setup

1. Import `main-workflow.json` into n8n (**Workflows > Import from file**).
2. Import the Error Alert workflow JSON the same way.
3. Create your own credentials in n8n: **Google Sheets**, **Gmail** and **Google Gemini (PaLM) API**.
4. Replace the placeholders in the nodes:
   - `YOUR_SHEET_ID`: your Google Sheet
   - `your-email@example.com`: where lead alerts and error alerts should arrive
   - `YOUR_API_KEY`: your Gemini API key, set through the n8n credential, never inside the workflow
5. Update the **Edit Fields** node with your own business name, sender name and business description.
6. In the main workflow, go to **Settings > Error Workflow** and select the Error Alert workflow.
7. Test with one sample row, then **Publish** both workflows.

> No API keys or personal data are included in this repository. Add your own credentials after importing.

## Sample test lead

| Field | Value |
|-------|-------|
| Name | Rahul Sharma |
| Message | I run a small clothing store and get 50+ WhatsApp enquiries daily. I want an AI agent to reply automatically. Budget around 30,000. Please call me today. |

Expected result: the lead is scored as high priority, a personalised reply is sent to the lead, and an alert reaches the owner.

## Roadmap

- WhatsApp Business Cloud alerts
- Status write-back to the Google Sheet
- Switch node to route leads by score
- Automatic follow-up emails
- Booking link in replies
- More lead sources (website forms, other channels)
- Weekly lead report
- Extra safety checks on AI output

## Contact

**Anup Tiwari**: AI agents, WhatsApp automation and website automation workflows.

Open to freelance and collaboration. Feel free to open an issue or connect through my GitHub profile.
