# Brands Shorts Automation

Overview
- Purpose: Generate short, shareable video scripts and media based on famous-brand prompts/templates for social shorts (TikTok, YouTube Shorts, Instagram Reels).
- Location: `Brands Shorts Automation/Famous_Brands_Shorts_Template.json` (n8n workflow export)

What this workflow does
- Accepts a topic or brand seed and generates:
  - Short, punchy script lines optimized for short-form video.
  - Visual prompt(s) for image or clip generation.
  - Suggested caption and hashtags.
- Optionally calls image/video generation APIs and returns media URLs or temporary hosted assets for downstream video assembly.

Key nodes (typical)
- Trigger node (webhook or schedule) — starts the flow with an input seed.
- LLM agent node — generates the short script, caption, hashtags, and image prompts (often enforced with a structured JSON output parser).
- Image/Video generation HTTP nodes — call image APIs or TTS/video renderers to create assets.
- Storage/upload nodes — upload media to tmpfile/S3 or return URLs.
- Output node — returns structured result or posts to a publishing queue (Google Sheets or social publish node).

Prompts & Output Schema
- Prompt pattern: short, brand-aware, attention-grabbing hooks (≤3 lines), a visual prompt for the image/clip, a caption (1-2 lines), and 3-8 hashtags.
- Example structured JSON expected from the agent:
  {
    "script": "",
    "visualPrompt": "",
    "caption": "",
    "hashtags": ["#example"]
  }

Credentials & Services
- LLM provider (OpenAI / OpenRouter / other)
- Image-generation API (Stability.ai, Midjourney proxy, DALL·E, etc.)
- Video/TTS provider (ElevenLabs, FAL.ai, VEED) if the workflow renders audio/video
- Storage: S3 / tmpfiles / other hosting for produced media
- Optional: Google Sheets (for queueing/tracking) or social API credentials (if publishing directly)

Testing guide
1. Connect required credentials in n8n and import the workflow.
2. Trigger with a sample brand seed (e.g., "Nike — quick product history hook") via the webhook or manual run.
3. Inspect the LLM node output and ensure it returns valid JSON matching the expected schema.
4. If enabled, verify image/video nodes return accessible URLs or binary outputs.
5. Review final output or appended sheet row for caption, hashtags, and media links.

Troubleshooting
- Agent returns malformed JSON: strengthen the output parser or tighten the system prompt to insist on JSON-only replies.
- Media uploads failing: check binary passthrough settings and multipart/form-data configuration on the webhook and HTTP nodes.
- Publishing errors: set publish nodes to draft mode or save outputs to Sheets while testing.

Production notes
- Replace tmpfile uploads with S3/R2 for production reliability and privacy.
- Add a manual approval step before publishing to avoid accidental brand misuse or copyright issues.
- Monitor API usage and rate limits for LLM and image/video providers.

Files
- Workflow: `Famous_Brands_Shorts_Template.json`

Last updated: 2025-12-31
