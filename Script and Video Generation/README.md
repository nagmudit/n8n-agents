# Script and Video Generation (Captions & Lipsync)

This workflow (`Script and Video Generation along with Captions and Lipsync.json`) automates short-form video production from a Telegram trigger: it detects an image + caption/theme, finds trending ideas, generates a short script, synthesizes speech with ElevenLabs, uploads audio/image to a public host, generates a lip-synced video via FAL/VEED queue, and saves metadata to Google Sheets.

## Quick Overview
- Trigger: `Telegram Trigger` (message/photo + caption)
- Trend research: `Search Trends with Perplexity` (Perplexity API)
- Script generation: `Generate Script with GPT-4` (OpenAI gpt-4o-mini/gpt-4 variants)
- TTS: `ElevenLabs Voice Synthesis` (ElevenLabs API)
- Audio upload: `Upload Audio to Public URL` (tmpfiles.org)
- Image hosting: `Build Public Image URL` (tmpfiles.org)
- Video generation: `FAL.ai Video Generation` → queued to VEED/FAL → `Wait for VEED` → `Download VEED Video`
- Caption generation: `Generate Caption with GPT-4`
- Save metadata: `Save to Google Sheets`

## Files
- `Script and Video Generation along with Captions and Lipsync.json` — n8n workflow file
- `README.md` — this document

## Key Config Variables (in `Workflow Configuration` node)
Set these in the `Workflow Configuration` (`set`) node near the start of the workflow:
- `elevenLabsApiKey` — your ElevenLabs API key
- `elevenLabsVoiceId` — ElevenLabs voice id to synthesize (voice path used in HTTP request)
- `falApiKey` — API key used to call the FAL / VEED queue endpoint
- `scriptMaxDuration` — maximum spoken duration (seconds), default 30
- `perplexityModel` — model to use for Perplexity (e.g., `sonar`)

## Environment variables / configuration placeholders
Use these as examples if you externalize keys or document them in your deployment:

- `ELEVENLABS_API_KEY=YOUR_ELEVENLABS_API_KEY`
- `ELEVENLABS_VOICE_ID=YOUR_VOICE_ID`
- `FAL_API_KEY=YOUR_FAL_API_KEY`
- `TELEGRAM_BOT_TOKEN=YOUR_TELEGRAM_BOT_TOKEN`
- `OPENAI_API_KEY=YOUR_OPENAI_API_KEY`
- `PERPLEXITY_API_KEY=YOUR_PERPLEXITY_API_KEY`

Note: credentials for Google Sheets should be configured inside n8n (OAuth2 credentials), not as plain env vars.

## Credentials & Services Required
- Telegram Bot token (to power `Telegram Trigger` and `Get Photo File from Telegram`) — configure Telegram nodes' credentials in n8n.
- OpenAI API key (gpt-4o-mini/gpt-4 variant) — configure in the `Generate Script with GPT-4` / `Generate Caption with GPT-4` nodes.
- ElevenLabs API key — used by `ElevenLabs Voice Synthesis` HTTP request. Place value in `Workflow Configuration`.
- FAL/VEED queue API key — used by `FAL.ai Video Generation` and polling `Download VEED Video`.
- Google Sheets OAuth2 — configured for `Save to Google Sheets` node.
- tmpfiles.org (public uploader) — used to host audio and images (HTTP request to `https://tmpfiles.org/api/v1/upload`). No credentials required for uploading there.
- Perplexity (Perplexity node) — if using Perplexity node ensure credentials or API access is configured in n8n.

## Prompts / System Messages (extracted)
These are the exact prompt templates used by the workflow.

- Search Trends (Perplexity) prompt:

  "Find the top 3 current viral trends related to: {{ caption }}. Focus on trending topics, hashtags, and content styles that are performing well on TikTok right now. Be specific and actionable. Limit your response strictly to 3 results only — no more."

- Generate Script (OpenAI) prompt:

  Based on these trends: {{ Search Trends output }}

  Create a viral 30-second maximum reel script about: {{ theme }}

  Requirements:
  - Maximum 30 seconds when spoken
  - Hook in first 2 seconds
  - Engaging and conversational
  - Optimized for voice synthesis
  - No special characters or formatting
  - Return ONLY the script text, nothing else

- ElevenLabs request body (TTS): JSON with `text`, `model_id`: `eleven_multilingual_v2`, and `voice_settings` (stability 0.5, similarity_boost 0.75). The node expects an audio/mpeg response and converts `.mpga` to `.mp3`.

- Generate Caption (OpenAI) prompt:

  Create an engaging reel caption for a video about: {{ theme }}

  Based on these trends: {{ Search Trends output }}

  Requirements:
  - Catchy hook in first line
  - Include 5-8 relevant trending hashtags
  - Keep it concise and engaging
  - Optimize for reel algorithm
  - Return ONLY the caption text with hashtags, nothing else

## Workflow Flow (high-level)
1. Telegram message arrives (image + optional caption).  
2. `Workflow Configuration` sets up keys and defaults.  
3. `Extract Photo and Theme` extracts `photoUrl` and `theme`.  
4. `Get Photo File from Telegram` downloads the photo binary.  
5. `Build Public Image URL` uploads the image to `tmpfiles.org` and returns a public URL.  
6. `Search Trends with Perplexity` finds 3 viral trends for the theme.  
7. `Generate Script with GPT-4` builds a concise script (<=30s).  
8. `ElevenLabs Voice Synthesis` synthesizes audio (mpga).  
9. `Convert .mpga to .mp3` normalizes audio binary.  
10. `Upload Audio to Public URL` uploads audio to `tmpfiles.org`.  
11. `FAL.ai Video Generation` is called with `image_url` + `audio_url` to create a lip-synced video via VEED pipeline.  
12. `Wait for VEED` pauses (10 minutes default) while VEED processes the job.  
13. `Download VEED Video` polls and downloads the final video.  
14. `Generate Caption with GPT-4` produces caption + hashtags.  
15. `Save to Google Sheets` logs metadata (idea, caption, URLs, status).

## Testing the Workflow (manual flow)
1. Configure all credentials in n8n.  
2. Activate the workflow.  
3. Send a photo to the Telegram bot with a caption or text describing the theme. Example: "beach workout" with an image.  
4. Monitor workflow execution in n8n — check each node's output.  
5. After completion the Google Sheet will have a new row with URLs to audio, image, and video.

## Example Telegram Input
- Photo: attach an image file to a message.  
- Caption/text: "viral beach workout" (this becomes `theme` used in prompts).

## Example Payloads & Endpoints
- tmpfiles upload (multipart/form-data) — used for both audio and images. Each upload returns a JSON with `data.url` which is used for downstream calls.

- FAL/VEED job POST body (example assembled by the `FAL.ai Video Generation` node):
```json
{
  "image_url": "<public image url>",
  "audio_url": "<public audio url>",
  "resolution": "480p"
}
```

- ElevenLabs TTS POST (node builds this body):
```json
{
  "text": "<script text>",
  "model_id": "eleven_multilingual_v2",
  "voice_settings": { "stability": 0.5, "similarity_boost": 0.75 }
}
```

## Notes & Gotchas
- The workflow expects ElevenLabs to return audio as `audio/mpeg` (mpga). The `Convert .mpga to .mp3` code node renames and sets the mime type for uploading.
- The FAL/VEED queue is polled after submitting a job — the `Wait for VEED` node is set to 10 minutes by default; adjust if your provider is faster or slower.
- `tmpfiles.org` is used as a convenient public uploader; replace with your preferred CDN or storage (S3, Cloudflare R2) for production-grade reliability.
- The Perplexity node is used for trend discovery; if you don’t have access, you can replace this with a different web-research tool or pass a simpler prompt directly to the OpenAI node.
- Ensure `Workflow Configuration` values are set before testing — the workflow uses these values at runtime.

## Troubleshooting
- Audio not playable: confirm `ElevenLabs` node returned audio and the `Convert .mpga to .mp3` node produced `audio_mp3` binary. Inspect the binary output in the node execution details.
- FAL/VEED job failed: check the POST response from `FAL.ai Video Generation` for errors and the `Download VEED Video` polling responses.
- Google Sheets row missing: ensure Google Sheets credentials are valid and `Save to Google Sheets` has correct document/sheet IDs.

## Customization Ideas
- Use S3 or another object store instead of `tmpfiles.org`.
- Add auto-posting to Instagram/TikTok via API or a scheduling tool.
- Make caption style adjustable (serious, humorous, promotional) via an input flag.
- Add optional video templates or overlays in the FAL/VEED job body.

---

**Last Updated:** December 30, 2025
**Status:** Draft

If you want, I can now:
- Scaffold a small test harness (Node script) to POST sample payloads to the Telegram webhook, or
- Replace `tmpfiles.org` uploads with S3 and add example bucket configuration.

---

## PDF excerpt (concise)
The original PDF included a step-by-step checklist and the same setup steps documented above. Key items required:
- Telegram Bot Token, ElevenLabs API Key + Voice ID, FAL/VEED API Key, Google Sheets, OpenAI API Key, Perplexity API Key.

Follow the sections above (Telegram setup, API configuration, AI processing setup, and voice/video generation) to get the workflow running.

## Example: Telegram message → expected Google Sheets row

Example Telegram message:
- Send a photo with caption: "fitness tips"

Expected Google Sheets row (example):

| IDEA | CAPTION | URL AUDIO | URL IMAGE | URL VIDEO | STATUS |
|------|---------|-----------|-----------|-----------|--------|
| fitness tips | Hook... #fitness #workout | https://tmpfiles.org/dl/123/audio.mp3 | https://tmpfiles.org/dl/123/image.jpg | https://tmpfiles.org/dl/123/video.mp4 | completed |

---

**Last Updated:** December 30, 2025
**Status:** Draft
