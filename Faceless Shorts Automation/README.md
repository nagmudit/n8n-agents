# Faceless Shorts Automation

Overview
- Purpose: Produce short “faceless” videos (scripts, captions, TTS audio, and render templates) using an LLM-driven pipeline and a Creatomate project template.
- Location: `Faceless Shorts Automation/Faceless_Shorts_System.json` and `Faceless Shorts Automation/Creatomate Template.txt`.

What the folder contains
- `Faceless_Shorts_System.json` — n8n workflow that accepts a topic/seed, generates a short-form script, creates captions, produces TTS audio, and prepares a video render payload using a templating service (Creatomate or equivalent).
- `Creatomate Template.txt` — example Creatomate project template or rendering instructions used by the workflow to assemble clips, overlays, captions, and timing.

High-level flow (typical)
1. Trigger (webhook / schedule / sheet row) receives a topic or content seed.
2. LLM agent generates:
   - Short script (hook + 2–4 lines)
   - Caption text and hashtags
   - Visual prompt(s) or asset selection instructions
   - Timestamps for on-screen text/captions
3. TTS node (ElevenLabs or other) generates voice audio from script.
4. Media nodes: image/video generation or selection, asset uploads, and assemble render payload.
5. Render call (Creatomate API or other video render API) uses `Creatomate Template.txt` plus generated assets to create final video.
6. Output: video URL or upload to storage and optional publish queue (Google Sheets / social API).

Prompts & Output Schema
- Agent typically returns structured JSON such as:
  {
    "script": "",
    "caption": "",
    "hashtags": [""],
    "tts_voice": "<voice id>",
    "visualPrompts": [""],
    "renderPayload": { /* fields matching Creatomate template */ }
  }
- `Creatomate Template.txt` contains placeholders and mapping notes for how the renderPayload fields map to the template.

Required Credentials & Services
- LLM provider (OpenAI / OpenRouter / other)
- TTS provider (ElevenLabs or chosen provider)
- Creatomate API key (or alternate render API) for templated video rendering
- Image generation API (optional) or media library storage (S3 / tmpfile)
- Google Sheets or storage for queues/logging (optional)

Testing & Validation
- Connect all credentials in n8n and import `Faceless_Shorts_System.json`.
- Run a manual test with a short seed prompt (aim for <30s script). Confirm these artifacts:
  - LLM output (script + structured JSON)
  - TTS audio file (playback)
  - Render payload assembled correctly per `Creatomate Template.txt`
  - Final render returns a video URL or stored binary
- Prefer draft mode or local storage for initial tests instead of direct publishing.

Troubleshooting
- Malformed JSON from LLM: tighten prompt to demand strict JSON and add a structured output parser node.
- TTS mismatch: verify chosen voice id and that audio file format matches render template requirements.
- Render failures: validate `renderPayload` fields against `Creatomate Template.txt` and test the template with static example payloads first.
- Binary handling: ensure nodes honor binary passthrough and multipart requirements when uploading audio/images to the render API.

Production Notes & Recommendations
- Use S3/R2 instead of tmpfile for production media hosting.
- Add a human approval step before publishing; faceless videos using brand content may require legal review.
- Monitor costs for TTS and render API calls; batch renders where possible.

Files
- Workflow: `Faceless_Shorts_System.json`
- Render template: `Creatomate Template.txt`

Last updated: 2025-12-31
