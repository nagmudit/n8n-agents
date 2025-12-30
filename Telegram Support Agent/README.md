# Telegram Support Agent

Overview
- **Purpose:** Provides a lightweight Telegram support chatbot powered by an LLM and a Google Sheets knowledge base.
- **Location:** `Telegram_Support_Agent/Telegram_Support_Agent.json` (n8n workflow export)
- **Trigger:** Telegram message updates via the `Telegram Trigger` node.

Architecture & Node Summary
- `Telegram Trigger` — listens for incoming messages.
- `Send a chat action` — sends typing indicator back to user.
- `AI Agent` (LangChain agent) — main orchestrator node. Prompted to read the incoming message, query the Google Sheets knowledge base, and return a structured JSON answer: `{ "answer": "" }`.
- `Google Gemini Chat Model` — LLM node (PaLM/Gemini) used as language model for the agent.
- `Get row(s) in sheet in Google Sheets` — reads the knowledge base (documentId points to the sheet). Cached result name: "Telegram Bot Knowledge Base".
- `Simple Memory` — session memory keyed by Telegram chat id.
- `Structured Output Parser` — enforces the agent returns strict JSON matching `{ "answer": "" }`.
- `Send a text message` — sends the `answer` field back to the Telegram chat.

Key Prompt / Behavior
- The `AI Agent` prompt instructs the agent to:
  1. Read the user message: `{{ $('Telegram Trigger').item.json.message.text }}`.
  2. Search the Google Sheets knowledge base for relevant items (questions, answers, keywords).
  3. Return a clear, human-like response based on the knowledge base or a polite fallback if nothing is relevant.
  4. Return ONLY valid JSON in the schema: `{ "answer": "<text>" }`.

Required Credentials & Permissions
- Telegram API credential configured in `Telegram Trigger` and Telegram nodes.
- Google Sheets OAuth2 credential for `Get row(s) in sheet in Google Sheets`.
- Google PaLM/Gemini API credential for `Google Gemini Chat Model` (optional: other LLMs may be substituted).

Environment & Configuration Notes
- The workflow references a specific Google Sheet ID: `1_niT7KNUGOyzhVZH6ylhnZ2SfPxJuGz9oWx_fb214g4` and `gid=0`. Replace with your own sheet if needed.
- The `AI Agent` expects the sheet to contain rows with searchable `question` / `answer` / `keywords` columns. Keep the knowledge base updated for best results.
- The `Simple Memory` node uses the Telegram chat id as `sessionKey` so brief chat history is preserved across messages.

Testing & Validation
- In n8n, enable the workflow and send a Telegram message to your bot.
- Inspect the `AI Agent` and `Structured Output Parser` node outputs to confirm the agent returns valid JSON, e.g. `{ "answer": "Here is the answer..." }`.
- If the `Structured Output Parser` fails, check the `AI Agent` node output for malformed text and adjust the system prompt to emphasize strict JSON output.

Troubleshooting
- No reply from bot: verify webhook registration for `Telegram Trigger` and that the `Telegram account` credential is valid.
- LLM errors or slow responses: check the `Google Gemini(PaLM) Api account` credential and rate limits; consider switching to a different LLM node (OpenAI/Anthropic) if needed.
- Knowledge base misses: ensure the Google Sheet has appropriate indexing columns and that `Get row(s) in sheet` node is configured to return relevant rows.

Security & Production Notes
- Do not commit credentials or API keys to source control. Use n8n credential management.
- For sensitive user data, review retention policies for `Simple Memory` and purge or encrypt memory in production.

How to customize
- Swap LLM: Replace `Google Gemini Chat Model` with an OpenAI or Anthropic model node and update the `AI Agent` language model mapping.
- Add fallback routing: If no answer is found, you can add a node to create a support ticket or notify a human operator.

Files
- Workflow: `Telegram_Support_Agent.json` (placed in this folder)

Last updated: 2025-12-31
