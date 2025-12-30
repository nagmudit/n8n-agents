# Agent Swarm

Overview
- Purpose: A multi-tool Telegram assistant that routes natural-language requests to specialized agent tools (email, calendar, contacts, web research, YouTube ideation) and logs actions.
- Location: `Agent Swarm/Agent Swarm.json` (n8n workflow export)

High-level Flow
- `Telegram Trigger` receives messages (text or voice). Voice messages are downloaded by `Download Voice File` and transcribed by `Transcribe Audio`; text messages are set via `Set 'Text'`.
- `Switch` routes input to `Input` which feeds `AI Agent` (router agent). The router's job is to call the appropriate tool (always use `Think` once to verify steps).
- `AI Agent` orchestrates calls to agent tools and returns a final `Response` to the Telegram chat.

Primary Tools / Agent Nodes
- `Email Agent` (agentTool) — sends emails, drafts, replies, labels, and reads via Gmail tool nodes (`Send Email`, `Create Draft`, `Email Reply`, `Get Emails`, `Get Labels`, `Label Emails`, `Mark Unread`).
- `Calendar Agent` (agentTool) — create/get/update/delete events via Google Calendar nodes (`Create Event`, `Create Event with Attendee`, `Get Events`, `Update Event`, `Delete Event`).
- `Contact Agent` (agentTool) — lookup and upsert contacts in Airtable (`Get Contacts`, `Add or Update Contact`).
- `Web Agent` (agentTool) — quick lookups via `Tavily`, deeper research via `Perplexity`, and weather via `OpenWeatherMap`.
- `YouTube Agent` (agentTool) — fetch video ideas (`YouTube Videos`) and append ideas to a Google Sheet (`Add Idea`, `Get Ideas`).

Support Nodes
- `Think` — tool the router uses to validate chosen actions.
- `Simple Memory` — memory buffer keyed by Telegram chat id to preserve short context.
- `Clean Up` (Code) — extracts intermediate steps and token usage, sums tokens.
- `Append row in sheet` — logs agent runs to an action log Google Sheet (input, output, actions, tokens, total_tokens).

LLM & Model Mapping
- Multiple OpenRouter model nodes (`GPT 4.1-mini*`) are mapped to different agent tools and the router agent.
- OpenAI (or chosen provider) used for transcription and any required LLM tasks where configured.

Key Prompts / Behavior
- Router `AI Agent` system prompt: route requests to the correct tool, call `Think` each time, and avoid doing final user-facing work itself (tools perform actions).
- Tool agent system messages enforce responsibilities and required pre-steps (e.g., `Get Emails` before `Email Reply`).

Credentials Required
- Telegram API credential (used by `Telegram Trigger` and Telegram response nodes).
- Gmail OAuth2 (send/read/label email nodes).
- Google Calendar OAuth2.
- Airtable API key (contacts table).
- Google Sheets OAuth2 (logging and YouTube ideas sheets).
- OpenRouter API key (LLM nodes) and OpenAI API key (transcription/embeddings if used).
- Tavily / Perplexity / OpenWeatherMap / Apify credentials for research and YouTube scraping nodes.

Testing & Validation
- Connect all credentials in n8n and enable the workflow (it is currently `active: false` by default).
- Test with a simple Telegram text message: confirm router chooses the expected tool, the tool is called, and a response is returned.
- Test voice: send a voice note to your bot; confirm `Download Voice File` → `Transcribe Audio` → router flow.
- Check the `Append row in sheet` outputs to confirm actions and token usage are logged.

Troubleshooting
- No response: verify Telegram webhook/credentials and that `Response` node uses correct chat id mapping.
- Wrong tool chosen: inspect `AI Agent` intermediateSteps output; refine the router system prompt or add stricter tool-selection examples.
- Missing contact/email IDs: ensure Airtable and Gmail nodes return expected fields; agent tools require `Get Emails`/`Get Contacts` pre-steps before follow-ups.

Security & Production Notes
- Never commit credentials. Use n8n credential management.
- Redact or avoid storing sensitive PII in logs; review retention on the action log sheet.
- Rate limits: watch LLM and API quotas (OpenRouter, Perplexity, Apify, Gmail limits).

Customization Ideas
- Add an approval step (human-in-the-loop) before executing destructive actions (deleting calendar events, sending emails).
- Swap transcription to ElevenLabs or another STT provider if needed for improved accuracy.

Files
- Workflow: [Agent Swarm/Agent Swarm.json](Agent%20Swarm/Agent%20Swarm.json)
- This README: [Agent Swarm/README.md](Agent%20Swarm/README.md)

Last updated: 2025-12-31
