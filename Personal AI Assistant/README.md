# Personal AI Assistant (Suite)

Collection of agent workflows that together provide a personal assistant: email, calendar, projects, research, and an orchestrating assistant.

## Contents
- `__Personal_Assistant_AI_Agent_2_0.json` — orchestrator agent that routes requests to tool-workflows (Email, Calendar, Projects, Research), provides Telegram integration, Slack tool, and a Knowledge Base vector store.
- `__Calendar_Agent.json` — calendar tool workflow: create events, create events with attendees, and list events via Google Calendar tool nodes.
- `__Email_Agent.json` — email tool workflow: send emails and fetch messages via Gmail tool nodes.
- `__Projects_Agent.json` — projects tool workflow: read/update project rows in a Google Sheet.
- `__Research_Agent.json` — research tool workflow: search Wikipedia, Hacker News, and SerpAPI and summarize results.

## High-level Purpose
- Provide an assistant that can accept natural-language requests (via Telegram or other triggers), decide which tool to call, and execute actions against Google Calendar, Gmail, Google Sheets, search APIs, Slack, and a vector Knowledge Base.

## Key Components & Nodes
- Orchestrator (`Personal Assistant` agent): system prompt instructs tool usage and orchestration; connected to `OpenAI Chat Model` (LM). It calls tool-workflows via `toolWorkflow` nodes and returns replies via `Telegram` nodes.
- Tool workflows: each tool (Calendar, Email, Projects, Research) exposes a `langchain.agent` node with a clear system prompt and one or more provider nodes (Google Calendar, Gmail, Google Sheets, SerpAPI, Wikipedia, HackerNews, Pinecone, etc.).
- Vector store & embeddings: `Embeddings OpenAI` + `Pinecone Vector Store` + `Knowledge Base` tool for company knowledge retrieval.
- Slack tool: `Send Slack Message` node to post messages to a channel when requested.

## Credentials & Setup Summary
- `OpenAI` / LM credential — required for agent / LM nodes (`OpenAi account 2` referenced).
- `Google Calendar` OAuth2 — required by `Calendar Agent` nodes (`Google Calendar account`).
- `Gmail` OAuth2 — required by `Email Agent` (`Gmail account`).
- `Google Sheets` OAuth2 — required by `Projects Agent` and Contacts tool (`Google Sheets account`).
- `SerpAPI` key — required by `Research Agent` if SerpAPI node is used.
- `Pinecone` API — required for `Pinecone Vector Store` and `Knowledge Base` tool if using vector retrieval.
- `Telegram` Bot credential — used by the orchestrator for Telegram Trigger/Reply.
- `Slack` OAuth2 — used by `Send Slack Message` node.

Recommended env vars / credential names:
- `OPENAI_API_KEY`, `GOOGLE_CALENDAR_CREDS`, `GMAIL_CREDS`, `GOOGLE_SHEETS_CREDS`, `PINECONE_API_KEY`, `SERPAPI_KEY`, `TELEGRAM_TOKEN`, `SLACK_OAUTH`.

## How to Use / Quick Start
1. Import all workflows into n8n and keep relative names intact.
2. Configure all credentials listed above in n8n (OpenAI, Google, Pinecone, SerpAPI, Telegram, Slack).
3. Enable the tool workflows (`Calendar Agent`, `Email Agent`, `Projects Agent`, `Research Agent`) so they are callable by the orchestrator via `toolWorkflow` nodes. Make sure the `workflowId` references inside each tool node match the imported workflows (they should if JSONs were imported together).
4. Start the orchestrator `Personal Assistant AI Agent 2.0` and test via its `Telegram Trigger` (send example requests) or by executing the workflow trigger node manually.

## Testing Tips
- Calendar: ask the orchestrator to "Schedule a 30-minute meeting tomorrow at 10am with alice@example.com" and verify Google Calendar receives an event. Use the `Get Events` tool to validate.
- Email: ask to "Send an email to bob@example.com with subject X" and verify a draft/send occurs in Gmail; use `Get Messages` to fetch recent emails.
- Projects: try "Update project Nimbus status to blocked" and verify the Google Sheet row is updated.
- Research: ask a factual query and verify the `Research Agent` searches Wikipedia first, then Hacker News, then SerpAPI.

## Troubleshooting
- Tool not callable: ensure the referenced `workflowId` in each `toolWorkflow` node matches the imported workflow's ID. If you re-imported, open the `toolWorkflow` node and re-select the workflow from the list.
- 401/403 errors: check credential scopes and refresh tokens for Google/Gmail/Calendar and ensure API keys are valid for SerpAPI/Pinecone.
- Wrong results from Knowledge Base: re-index embeddings into Pinecone and confirm `Embeddings OpenAI` credentials are valid.
- Telegram messages not delivered: confirm the bot token and webhook/polling settings.

## Security & Production Notes
- Never commit secrets; store credentials in n8n's credential store or environment.
- For production, restrict who can trigger the orchestrator (e.g., limit Telegram chat IDs, require auth).
- Add logging, rate limiting, and human approval steps for destructive actions (deleting calendar events, sending high-impact emails).

---
Files: __Personal_Assistant_AI_Agent_2_0.json, __Calendar_Agent.json, __Email_Agent.json, __Projects_Agent.json, __Research_Agent.json
