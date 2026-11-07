# Bengaluru Fresher Job Alerts

An n8n workflow that searches LinkedIn, Naukri, and Hirist for entry-level jobs in Bengaluru, checks each posting against a candidate profile with Groq, and sends a tailored alert to Telegram.

The workflow export is [`fresher-job-alerts-v2.n8n.json`](./fresher-job-alerts-v2.n8n.json).

## What it does

1. Starts when you manually execute the workflow in n8n.
2. Builds search requests for the configured keywords on LinkedIn, Naukri, and Hirist.
3. Parses the results and filters for Bengaluru-area roles, excluding senior titles and postings whose stated experience is above the configured fresher-friendly limits.
4. Removes duplicate company/title combinations within the search and jobs already seen in previous workflow executions.
5. Fetches LinkedIn job descriptions where available and checks descriptions for explicit experience requirements.
6. Sends each remaining job to Groq for a profile fit summary, then formats and sends the result to Telegram, one job at a time with a five-second pause.

The default maximum is 30 alerts per run. Job search results and job descriptions depend on what each source makes available at execution time.

## Requirements

- An n8n instance that supports the node types and versions in the exported workflow.
- A Groq API key.
- A Telegram bot token and the destination chat ID.
- Network access from n8n to the job sites, Groq, and Telegram.

## Setup

1. In n8n, import `fresher-job-alerts-v2.n8n.json`.
2. Create an **HTTP Header Auth** credential for the **Groq Summarise** node:
   - Header name: `Authorization`
   - Header value: the `Bearer` scheme followed by your Groq API key. Keep the key in the n8n credential; do not commit it.
3. Create a **Telegram** credential for the **Send to Telegram** node using your bot token.
4. Open the **Config** node and replace `YOUR_TELEGRAM_CHAT_ID` with the destination chat ID.
5. Review the candidate profile, search keywords in **Build Queries**, and settings in **Config**. Adjust them for your needs.
6. Save the workflow and select **Execute workflow** to run it.

The exported workflow contains credential placeholders, not usable credentials. Credentials must be configured in your own n8n instance.

## Configuration

The **Config** node includes:

| Setting | Default | Description |
| --- | --- | --- |
| `telegramChatId` | `YOUR_TELEGRAM_CHAT_ID` | Telegram chat or group to receive alerts |
| `postedWithinHours` | `24` | How recently a job should have been posted |
| `maxJobsPerRun` | `30` | Maximum jobs analyzed and sent in one run |
| `groqModel` | `llama-3.3-70b-versatile` | Groq chat-completion model |
| `candidateProfile` | Example fresher profile | Profile used to assess fit and suggest resume keywords |

Edit the `keywords` array in the **Build Queries** node to change the searches. The location and experience checks are implemented in the **Filter Bengaluru Freshers** and **Experience Check** nodes.

## Limitations

- LinkedIn results use its public guest endpoint. Naukri and Hirist use unofficial endpoints; these may change, rate-limit requests, or return CAPTCHA/403 responses. Search requests are configured to continue on errors, so a source can fail while the rest of the workflow proceeds.
- Search results and descriptions can be incomplete or unavailable. Groq's fit assessment is only as useful as the posting data it receives; verify details on the original job listing.
- Previously seen jobs are tracked by n8n's **Remove Already-Sent Jobs** node at workflow scope, with a history size of 20,000. This depends on the n8n instance retaining workflow execution data.
- Groq usage is subject to its availability, model access, and rate limits. The workflow processes jobs individually and waits five seconds between Telegram sends.
- This workflow is manual-triggered; it does not run on a schedule unless you add and configure a schedule trigger in n8n.

## Privacy

The candidate profile and job posting details are sent to Groq for analysis. Configure the profile accordingly and review the service's data handling terms before use. API credentials are stored in n8n credentials and should not be added to this repository.
