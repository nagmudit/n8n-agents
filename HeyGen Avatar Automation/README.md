# HeyGen Avatar Automation

A compact n8n workflow to generate avatar videos with HeyGen using a scriptwriter agent (convert newsletter text into a short spoken script, then render an avatar video).

## Purpose
- Produce short avatar videos via HeyGen by combining a script generation step (AI) with HeyGen's video generation and polling endpoints.

## Key Nodes
- `When clicking ‘Test workflow’` — manual trigger used for testing.
- `News` — (optional) fetches source content via Apify to feed the scriptwriter.
- `Script Writer` — agent node that turns input text into a concise spoken script (system prompt targets 10–20s scripts).
- `GPT 4.1 Mini` — language model credential (OpenRouter) used by the `Script Writer` agent.
- `Generate Video` / `Generate Video1` — POST to `https://api.heygen.com/v2/video/generate` to create videos (body includes `avatar_id`, `voice_id`, and `input_text`).
- `Get Avatars` / `Get Voices` — helper HTTP requests to list available avatars and voices (use to get IDs for the generator).
- `Get Video` / `Get Video1` — poll `video_status.get` to check generation status and obtain final asset URLs.
- `30 Seconds` / `Wait` — wait nodes used to poll until the video status becomes `completed`.

## Credentials & Setup
- Create an HTTP Header Auth credential in n8n for HeyGen and attach it to the HeyGen nodes (`Generate Video`, `Get Video`, `Get Avatars`, `Get Voices`).
  - Typical header: `Authorization: Bearer <YOUR_HEYGEN_API_KEY>` and `accept: application/json`.
- Configure `OPENROUTER_API_KEY` in n8n for the `GPT 4.1 Mini` node if using OpenRouter.
- If using the `News` node, add your `APIFY_TOKEN` credential.

Recommended environment variables / credential names:
- `HEYGEN_API_KEY` — HeyGen API key (store in n8n credential store)
- `OPENROUTER_API_KEY` — for language model calls
- `APIFY_TOKEN` — if using Apify for content scraping

## How to Use
1. Import or open the workflow in n8n.
2. Configure the `HeyGen` HTTP Header Auth credential and the LM credential.
3. (Optional) Run `Get Avatars` and `Get Voices` to retrieve `avatar_id` and `voice_id` values.
4. Update the `jsonBody` in `Generate Video`/`Generate Video1` to set the desired `avatar_id`, `voice_id`, and `input_text` (the agent fills `input_text` when using `Script Writer`).
5. Execute the workflow (manually trigger or let `News` feed it). The workflow will call the generate endpoint, then poll `video_status.get` until `status == completed`.
6. Inspect the `Get Video` node output for the final download URL or result payload.

## Usage Example
- Quick test: run `Get Avatars` → pick an `avatar_id`; run `Get Voices` → pick `voice_id`; set `input_text` to `Hello from n8n` and execute `Generate Video`.

Sample `jsonBody` for `Generate Video` (update `avatar_id`, `voice_id`, and `input_text`):
```json
{
  "video_inputs": [
    {
      "character": { "type": "avatar", "avatar_id": "<AVATAR_ID>", "avatar_style": "normal" },
      "voice": { "type": "text", "input_text": "Hello from n8n.", "voice_id": "<VOICE_ID>", "speed": 1.0 }
    }
  ],
  "dimension": { "width": 1280, "height": 720 }
}
```

## Testing & Validation
- Use `When clicking ‘Test workflow’` to run an end-to-end test.
- Verify the `Generate Video` node returns a `video_id` in its `data` object.
- Confirm `Get Video` / `Get Video1` returns `status: completed` and includes the final asset URL.

## Troubleshooting
- 401/403 errors: confirm HeyGen API key and header format.
- `status` never becomes `completed`: increase polling intervals, or confirm HeyGen returned a valid `video_id`.
- Long generation times: use smaller video `dimension` or shorter `input_text` for faster tests.
- Rate limits: batch small tests and respect provider rate limits.

## Notes & Recommendations
- Store API keys in n8n credentials (do not inline secrets in the workflow JSON for production).
- For CICD or production, replace manual triggers with webhook or queue-based triggers and persist final assets to S3 or cloud storage.
- Respect HeyGen usage policies and voice cloning consent when using uploaded/third-party voices.

---
Workflow file: HeyGen_Avatar.json
