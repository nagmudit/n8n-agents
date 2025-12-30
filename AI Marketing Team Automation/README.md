# AI Marketing Team Automation

Overview
- Purpose: A collection of n8n workflows for marketing automation: blog writing, image generation/editing, faceless video creation, LinkedIn posting, and a top-level orchestration workflow.
- Location: This folder contains several exported workflows (JSON) that chain LLMs, image/video APIs, and Google Sheets for content production and publishing.

Files in this folder
- `AI_Marketing_Team.json` — Orchestrator that can route tasks between the sub-workflows (triggering blog posts, images, videos, and LinkedIn posting pipelines).
- `Blog_Post.json` — Generate article drafts from prompts or topic seeds, optionally output HTML or Markdown and append to Google Sheets / CMS.
- `Create_Image.json` — Use an image-generation model (Stable Diffusion / API) to create visuals from prompts; returns URLs or binary images.
- `Edit_Image.json` — Image editing pipeline (inpainting, resizing, overlays) that accepts an input image and edit prompt.
- `Faceless_Video.json` — Create short faceless videos: script → TTS → clip assembly → captions; integrates with TTS (ElevenLabs or similar) and a video render API.
- `LinkedIn_Post.json` — Generate LinkedIn post copy and image prompts from content inputs, optionally post via LinkedIn node or save drafts to Sheets.
- `Search_Images.json` — Search/curation helper: query image repositories or APIs and return candidate assets.

Common patterns & components
- LLM Nodes: prompt-driven agents for drafting copy, creating titles, and producing captions.
- Image/Video Tools: HTTP / vendor API nodes for image generation, editing, TTS, and video creation (ElevenLabs, Apify, FAL/VEED, or other providers).
- Storage & Tracking: Google Sheets for tracking ideas, outputs, and publishing status; tmpfile/S3 used for temporary hosting in some pipelines.
- Output Validation: structured JSON output parsers are used in some flows to ensure predictable downstream fields (title, body, mediaUrl, publishStatus).

Required Credentials (examples)
- OpenAI / OpenRouter / other LLM API key
- ElevenLabs (TTS) API key (optional)
- Image generation API key (Stability.ai, Midjourney via proxy, or other provider)
- Video render API (FAL.ai, VEED, or custom renderer) credentials
- Google Sheets OAuth2 for logging and content sinks
- LinkedIn API credentials for automated posting (optional)

Basic testing steps
1. Connect credentials in n8n and import/enable the workflow you want to test.
2. Start with `Search_Images.json` or `Create_Image.json` using a sample prompt to validate the image provider response.
3. Test `Blog_Post.json` with a single topic seed and confirm text quality and structured output (title, body, tags).
4. For `Faceless_Video.json`, run with a short script (<45s) and confirm TTS audio, upload, and render job flow.
5. For publishing flows (`LinkedIn_Post.json`), set posting to draft mode first or save outputs to Google Sheets for manual review before enabling live posting.

Troubleshooting & tips
- Use drafts: switch posting nodes to `createDraft` where possible while testing to avoid accidental public posts.
- Rate limits & cost: watch LLM/image/video API quotas; prefer smaller models for iterative testing.
- Binary handling: ensure webhooks and HTTP nodes support `multipart/form-data` or binary passthrough when transferring images/audio between nodes.
- Replace tmpfile uploads with S3 or R2 for production to ensure reliability and privacy.

Customization ideas
- Add a human-approval step (Slack or email) before publishing LinkedIn or video content.
- Add A/B title generation in `Blog_Post.json` and test performance via Google Sheets metrics.
- Build a simple dashboard that reads the Google Sheets tracking rows and shows status, links, and publish history.

Files
- Workflow files: See the JSON exports in this folder for exact node configs.

Last updated: 2025-12-31
