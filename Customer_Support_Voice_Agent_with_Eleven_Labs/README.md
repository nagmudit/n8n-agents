# Customer Support Voice Agent with Eleven Labs

Overview
- **Purpose:** Handle short, factual customer support answers from call transcriptions using an LLM and a company knowledge base (Google Sheets).
- **Location:** `Customer_Support_Voice_Agent_with_Eleven_Labs/Customer_Support_Voice_Agent_with_Eleven_Labs.json`
- **Trigger:** `Webhook` (POST) — receives call transcription payloads (expects `body.searchQuery`).

Node Summary
- `Webhook` — entrypoint; path: `763e88ba-1900-4050-bbf9-72c819b2e95e`.
- `AI Agent` (LangChain agent) — system prompt directs behavior: handle only the last request, call `Get Knowledgebase` once, answer ≤3 short sentences/bullets, ask one clarifying question if needed, and avoid fabrications or internal tool mentions.
- `Google Gemini Chat Model` — LLM mapped as the agent's language model (replaceable).
- `Get row(s) in sheet in Google Sheets` — knowledge base lookup (sheet id `1V0nRa5MZhtANSBvCGmCLA5tIPxaxt2AzD7Y18JKl8C4`, `gid=0`).
- `Think` — tool for short internal reasoning steps used by the agent.
- `Respond to Webhook` — sends the agent's reply back as the webhook response.

System Prompt Highlights
- Call `Get Knowledgebase` exactly once per request to fetch minimal relevant data.
- Refer to data as “database” or “base de connaissances” — never mention sheets/tools/nodes.
- If data missing: respond in French: “L’information n’est pas disponible dans la base de connaissances.”
- Keep answers concise (≤3 sentences/bullets) and professional.
- Do not attempt scheduling/booking or call tools other than `Get Knowledgebase`.

Required Credentials
- n8n webhook must be reachable (configure `Webhook` node and n8n public URL).
- Google Sheets OAuth2 credential for the knowledge base node.
- Google PaLM/Gemini API credential for the `Google Gemini Chat Model` node.

Configuration Notes
- Replace the Google Sheet ID with your own knowledge base if needed.
- Ensure incoming webhook payload includes `body.searchQuery` with the transcription or last user utterance.

Testing
- Send a POST to the webhook URL with JSON body containing `searchQuery`:

```bash
curl -X POST https://<your-n8n-host>/webhook/763e88ba-1900-4050-bbf9-72c819b2e95e \
  -H "Content-Type: application/json" \
  -d '{"searchQuery":"Is product X refundable?"}'
```

- Inspect `AI Agent` and `Get row(s) in sheet in Google Sheets` node outputs for fetched database rows and the final reply.

Troubleshooting
- Empty or wrong reply: check that the `Get row(s) in sheet` node returns relevant rows and the sheet columns match expected schema.
- No webhook response: ensure n8n is reachable and the webhook is active; check `Respond to Webhook` node output.
- LLM issues: verify PaLM/Gemini credential and quota; swap to another LLM node if needed.

Security & Production
- Do not commit credentials. Use n8n credential storage.
- Sanitize or redact PII from transcriptions before storing or logging.
- Review retention rules for transcripts and agent memory.

Customization
- Swap `Google Gemini Chat Model` with OpenAI/Anthropic if preferred.
- Add a TTS step (ElevenLabs) before responding if you need audio playback; integrate ElevenLabs API node and return audio or URL in the webhook response.

Files
- Workflow: `Customer_Support_Voice_Agent_with_Eleven_Labs.json`

Last updated: 2025-12-31
