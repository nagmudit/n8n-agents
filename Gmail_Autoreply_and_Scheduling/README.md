# Gmail Autoreply and Scheduling

This n8n workflow automates email replies and scheduling based on incoming Gmail messages. It uses an AI agent to parse incoming messages and decide whether to reply or schedule a meeting.

Files
- `Gmail_Autoreply_and_Scheduling.json` — n8n workflow
- `README.md` — this document

Quick overview
- Trigger: `Gmail Trigger` (polls every minute)
- AI agent: `AI Agent` (LangChain agent with Google Gemini model)
- Output parsing: `Structured Output Parser` to enforce JSON responses
- Actions: `Reply to a message` (Gmail) or `Create an event` (Google Calendar)

Primary prompt (agent)
The agent prompt (defined in the workflow) instructs the AI to:
- Use the provided personal context (law student, backend developer) when responding
- Classify the incoming email as either `reply` or `schedule_meeting`
- Return a JSON object with these fields: `action`, `replyText`, `meetingTimeStart`, `meetingTimeEnd`, `messageId`

Key nodes and behavior
- `Gmail Trigger`: polls Gmail and forwards incoming messages to the agent
- `AI Agent`: receives the email content and returns structured JSON (action + content)
- `Structured Output Parser`: validates/parses the agent's JSON output
- `If`: routes to either `Reply to a message` or `Create an event`
- `Reply to a message`: sends the reply via Gmail
- `Create an event`: creates the calendar event on the configured Google Calendar

Credentials required
- Gmail OAuth2 for Gmail Trigger and Gmail node
- Google Calendar OAuth2 for `Create an event`
- Google PaLM/Gemini credentials for the `Google Gemini Chat Model` node (used by the agent)

Setup
1. Create or import the workflow into your n8n instance.
2. Configure credentials:
   - Gmail OAuth2 credential for `Gmail Trigger` and `Reply to a message` nodes
   - Google Calendar OAuth2 credential for `Create an event` node
   - Google PaLM/Gemini API credential for the language model node
3. Review the `AI Agent` prompt in the workflow and update personal context if needed.
4. Activate the workflow.

Testing
- Send a test email to the Gmail account with a clear scheduling request (e.g., "Can we meet next Tuesday at 3pm?").
- Observe the workflow execution in n8n and verify whether it selects `Create an event` and creates the calendar entry.
- Send a non-scheduling query and verify it replies with a professional, concise response.

Output format expected from the agent
The agent must return JSON like:
```
{
  "action": "reply" OR "schedule_meeting",
  "replyText": "Full reply text to send",
  "meetingTimeStart": "YYYY-MM-DDTHH:MM:SSZ (if scheduling)",
  "meetingTimeEnd": "YYYY-MM-DDTHH:MM:SSZ (if scheduling)",
  "messageId": "(message id to reply to)"
}
```

Troubleshooting
- If replies are not sent: check Gmail OAuth2 credential and the `Reply to a message` node mapping.
- If events are not created: ensure `meetingTimeStart` and `meetingTimeEnd` are present and the Google Calendar credential is valid.
- If the agent returns invalid JSON: inspect the `AI Agent` execution and the `Structured Output Parser` output for errors.

Security & privacy
- The workflow uses OAuth2 credentials stored in n8n; do not commit credentials to source control.
- The agent prompt includes personal context — review before sharing the workflow.

Last updated: December 31, 2025
