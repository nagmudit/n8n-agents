# LinkedIn Automation

This n8n workflow (`LinkedIn_Automation.json`) automates LinkedIn post creation from rows in a Google Sheet. It reads pending rows, generates post copy and an image prompt via an AI agent, fetches a generated image, posts to LinkedIn, and updates the sheet status.

Files
- `LinkedIn_Automation.json` — n8n workflow
- `README.md` — this document

Quick overview
- Trigger: `Schedule Trigger` — runs on a schedule to scan the Google Sheet
- Source: `Get row(s) in sheet` — reads rows from a Google Sheet (documentId configured)
- Filter: `If` — selects rows where `Status` == `Pending`
- Rate-limit: `Limit` — controls how many rows are processed per run
- AI agent: `AI Agent` (LangChain agent with Google Gemini) — generates LinkedIn post and `image_prompt`
- Output parsing: `Structured Output Parser` — enforces JSON output from the AI agent
- Image: `HTTP Request` — calls an image generation endpoint using the `image_prompt`
- LinkedIn: `Create a post` — posts content (with image) via LinkedIn node
- Sheet updates: `Append or update row in sheet` / `Append or update row in sheet1` — write back status and generated content

Primary prompt (agent)
The `AI Agent` prompt instructs the model to:
- Generate a 100–200 word LinkedIn post for the title in `LinkedIn Post Title`.
- Include a hook, concise paragraphs, optional emojis, and an ending CTA.
- Produce a short, 1–2 sentence image prompt (no text in image) suitable for an AI image generator.
- Return results strictly as JSON with fields: `title`, `post`, `image_prompt`.

Credentials required
- Google Sheets OAuth2 — to read/write rows
- Google PaLM/Gemini (Google API) — for the AI model node
- LinkedIn OAuth2 — to create posts via the `Create a post` node

Setup
1. Create/import the workflow into your n8n instance.
2. Configure credentials:
   - Google Sheets OAuth2 for `Get row(s) in sheet` and sheet write nodes
   - Google PaLM/Gemini API credential for `Google Gemini Chat Model` node
   - LinkedIn OAuth2 credential for `Create a post` node
3. Configure the `Get row(s) in sheet` node to point to your copy of the Google Sheet (documentId and sheetName).
4. Adjust the schedule on `Schedule Trigger` to your desired cadence.
5. Optionally change the `Limit` node to control posts per run.
6. Activate the workflow.

Testing
- Add a test row to the configured Google Sheet with `Status` set to `Pending` and a `LinkedIn Post Title` value.
- Run the schedule trigger (or execute manually) and observe the workflow.
- Check that the LinkedIn post appears in the `Create a post` node's execution output and that the sheet status updates to `Posted`.

Expected AI output format
```
{
  "title": "<LinkedIn Post Title>",
  "post": "<Generated LinkedIn post text>",
  "image_prompt": "<short image prompt>"
}
```

Troubleshooting
- No rows processed: verify `Get row(s) in sheet` documentId/sheetName and that rows have `Status` == `Pending`.
- AI returns invalid JSON: check `AI Agent` execution and the `Structured Output Parser` output.
- Image generation failures: verify the `HTTP Request` URL template and API availability.
- LinkedIn posting errors: re-check LinkedIn OAuth2 credentials and `Create a post` node configuration.

Privacy & security
- OAuth2 credentials are stored in n8n — do not commit secrets.
- Review generated content before posting publicly if brand safety is a concern.

Customization ideas
- Add sentiment or tone controls to the prompt (e.g., professional, humorous).
- Add scheduling/publishing time columns in the sheet to control when posts go live.
- Use a dedicated image generation service (Gemini) with higher fidelity.

Last updated: December 31, 2025
