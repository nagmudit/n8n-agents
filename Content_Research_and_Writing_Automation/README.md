# Content Research and Writing Automation

This n8n workflow (`Content_Research_and_Writing_Automation.json`) automates keyword discovery, content idea generation, and article drafting using web suggestions and AI. It runs on a schedule, fetches search suggestions, generates content ideas, and writes SEO-optimized draft articles into Google Sheets.

## Quick overview
- Triggers: `Schedule Trigger` nodes (runs on schedule)
- Keyword discovery: `HTTP Request` to Google Suggest → `Code in JavaScript` parses top keywords
- Idea generation: `AI Agent1` (LangChain agent) suggests 3 content ideas per topic
- Draft generation: `AI Agent3` (LangChain agent) produces 450–650 word SEO-optimized articles
- Outputs: `Append row in sheet` writes ideas and drafts to Google Sheets

## Files
- `Content_Research_and_Writing_Automation.json` — n8n workflow
- `README.md` — this document

## Key prompts (extracted)
- Keyword fetch: Google Suggest endpoint (`suggestqueries.google.com/complete/search?client=firefox`) returns search suggestions for a query.

- Idea generation prompt (AI Agent1):
  "Suggest 3 trending and SEO-optimized content ideas based on the main query and related keywords. Each idea should include a catchy title and a 1–2 line description. Return strict JSON with `main_query` and `ideas` array."

- Article drafting prompt (AI Agent / AI Agent3):
  "Write a comprehensive, SEO-optimized blog article (450–650 words) for the provided main topic and title. Include introduction, 3–4 subheadings, conclusion, meta title (<=60 chars), meta description (<=150 chars), 5 meta tags, and 5 trending hashtags. Return strict JSON with fields for `content`, `meta_title`, `meta_description`, `meta_tags`, and `hashtags`."

## Key nodes and behavior
- `Schedule Trigger` / `Schedule Trigger1`: start the pipeline on a configured schedule.
- `HTTP Request` / `HTTP Request1`: query Google Suggest for keywords.
- `Code in JavaScript` nodes: parse raw HTTP response and extract top suggestions.
- `AI Agent1` / `AI Agent2`: produce content idea lists (3 ideas).
- `Loop Over Items` / `SplitInBatches` nodes: iterate over ideas.
- `AI Agent3`: generate final article drafts for each idea.
- `Append row in sheet`: write results to Google Sheets for review and publishing.

## Credentials & services required
- Google Sheets OAuth2 — to append results to a sheet
- Google PaLM/Gemini (Google API) — for the AI model nodes
- (Optional) any third-party SEO or keyword APIs if you replace Google Suggest

## Setup
1. Import the workflow into your n8n instance.
2. Configure credentials:
   - Google Sheets OAuth2 credential for `Append row in sheet` node
   - Google PaLM/Gemini API credential for the `Google Gemini Chat Model` nodes
3. Set the schedule on the `Schedule Trigger` nodes to the desired frequency.
4. Adjust the `HTTP Request` query parameter (`q`) to the seed keywords you want to scan (e.g., `fruits`, `cars`).
5. Activate the workflow.

## Testing
- Manually run the `HTTP Request` and `Code in JavaScript` nodes to confirm keyword extraction works.
- Trigger the `AI Agent1`/`AI Agent3` nodes with sample inputs to validate JSON outputs.
- Verify rows are appended to the configured Google Sheet with expected fields: `main_query`, `idea_title`, `article_title`, `content`, `meta_title`, `meta_description`, `meta_tags`, `hashtags`.

## Troubleshooting
- No suggestions returned: inspect the `HTTP Request` node output and ensure the query parameter `q` is present and correct.
- AI returns invalid JSON: check the agent execution and `Structured Output Parser` nodes for errors.
- Sheets writes fail: verify Google Sheets credential and correct documentId/sheetName.

## Customization ideas
- Replace Google Suggest with an SEO API (Ahrefs, SEMrush) for richer keyword data.
- Add readability and plagiarism checks (e.g., integrate a plagiarism API or local checks).
- Auto-schedule publishing by connecting to a CMS or scheduling tool.

---

**Last updated:** December 31, 2025
