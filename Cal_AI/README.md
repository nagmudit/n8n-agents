# Cal AI

Overview
- Purpose: Analyze food images and return a structured JSON nutritional breakdown (calories, protein, carbs, fat) per item and totals.
- Location: [Cal_AI/Cal_AI.json](Cal_AI/Cal_AI.json)
- Triggers: two webhooks (`Webhook` and `Webhook1`) accepting POST requests with image binary data.

Node Summary
- `Webhook` / `Webhook1` — entrypoints; configured with paths `f22ecd48-d6f9-48bf-9c7a-8bfbf071cf59` and `3477492c-c3a1-4333-9a04-b3abfed82566`.
- `AI Agent` / `AI Agent1` — LangChain agent nodes expecting an image (passthroughBinaryImages enabled on the first agent) and a prompt that requires strict JSON output.
- `Google Gemini Chat Model` / `Google Gemini Chat Model1` — LLM nodes mapped as the agents' language models (replaceable).
- `Structured Output Parser` / `Structured Output Parser1` — enforce output conforms to the required JSON schema.

Prompt Notes
- The agent prompt asks for a detailed nutritional breakdown and enforces exact JSON structure:

{
  "status": "success",
  "food": [ { "name":"","quantity":"","calories":0,"protein":0,"carbs":0,"fat":0 } ],
  "total": { "calories":0,"protein":0,"carbs":0,"fat":0 }
}

- First `AI Agent` has `passthroughBinaryImages: true` set to allow image binaries through to the model.

Required Credentials & Inputs
- n8n webhook endpoint must be reachable and accept multipart/form-data with image binary.
- Google PaLM/Gemini API credential for LLM nodes.

Testing
- Example cURL (replace `<your-n8n-host>` and `PATH`):

```bash
curl -X POST "https://<your-n8n-host>/webhook/f22ecd48-d6f9-48bf-9c7a-8bfbf071cf59" \
  -F "file=@/path/to/food.jpg"
```

- Check the `Structured Output Parser` node to confirm JSON validity and numeric estimates.

Troubleshooting
- Missing image: ensure the request uses `multipart/form-data` and the file field matches the webhook node configuration.
- Invalid JSON from agent: strengthen the system prompt to emphasize strict JSON-only responses and validate sample outputs.
- LLM failures: verify PaLM/Gemini credentials and quota; consider swapping LLM if reliability issues persist.

Security & Notes
- Estimates are heuristic; warn users they are approximate and not a substitute for professional nutrition analysis.
- Do not store raw images with sensitive PII without consent.

Files
- Workflow: [Cal_AI/Cal_AI.json](Cal_AI/Cal_AI.json)

Last updated: 2025-12-31
