# Research and Content Creation Agents

Automated content pipeline: trigger from Google Sheets, search the web (Tavily), aggregate results, and produce blog posts, LinkedIn posts, and X (Twitter) posts using LLM agents.

## Purpose
- Turn a content brief (from Google Sheets) into multi-format content: blog (two paragraphs), LinkedIn post, and X tweet. The workflow searches the web for relevant articles, aggregates raw content, then runs specialized agent nodes to create each format and updates the campaign row in Google Sheets.

## Key Nodes
- `Google Sheets Trigger` — fires when a new row (campaign) is added to the Content Creation sheet.
- `Set Search Fields` — maps sheet columns (`Content Subject`, `Target Audience`) into workflow variables.
- `Search Internet` — HTTP POST to `https://api.tavily.com/search` returning top results and raw content.
- `Split Out` / `Aggregate` — split and re-aggregate search results, keeping `title` and `raw_content` fields for the writers.
- `Blog Writer` — LLM agent node that writes a two-paragraph blog article from aggregated content.
- `LinkedIn` — LLM agent that crafts a LinkedIn post tailored to the target audience (line breaks, emojis, hashtags).
- `X` — LLM agent that crafts a concise tweet optimized for engagement.
- `Update Campaign` — writes outputs back to the Google Sheet (`Blog`, `LinkedIn`, `X`) for the campaign.
- `OpenAI Chat Model` nodes — LLM provider nodes referenced by agents (OpenAI credentials required).

## Credentials & Setup
- `Google Sheets Trigger` credential — must have access to the `Content Creation` sheet (documentId referenced in node).
- `Tavily API key` — replace `YOUR-API-KEY-HERE` in the `Search Internet` node or store as an environment/credential and reference it in the node body.
- `OpenAI` credential — used by `OpenAI Chat Model` nodes for agent text generation.
- `Google Sheets` OAuth2 — used by `Update Campaign` node to write back generated outputs.

Suggested environment variables / credential names:
- `TAVILY_API_KEY`, `OPENAI_API_KEY`, `GOOGLE_SHEETS_CREDS`.

## Flow Overview
1. A new campaign row is added to the `Content Creation` Google Sheet.
2. `Google Sheets Trigger` fires and `Set Search Fields` extracts the `Content Subject` and `Target Audience`.
3. `Search Internet` calls Tavily to find relevant articles and returns `raw_content` and metadata.
4. Results are `Split Out` and then `Aggregate`d to produce a combined content payload for the writer agents.
5. `LinkedIn` and `X` agents transform the aggregated content into platform-optimized posts; `Blog Writer` produces a two-paragraph article.
6. `Update Campaign` appends the generated `Blog`, `LinkedIn`, and `X` outputs back to the sheet.

## Testing
- Add a sample row to the referenced Google Sheet with `Content Subject` and `Target Audience`.
- Watch the workflow execute in n8n: `Search Internet` should return results, and the `Blog Writer`, `LinkedIn`, and `X` nodes should produce outputs.
- Confirm the `Update Campaign` node writes the results into the corresponding sheet row.

## Troubleshooting
- No search results: verify Tavily API key and query formatting.
- Poor content quality: tune the agents' system prompts inside `Blog Writer`, `LinkedIn`, and `X` nodes for tone/length.
- OpenAI errors: check `OPENAI_API_KEY` and quota/limits.
- Google Sheets write fails: confirm `Update Campaign` credentials and that the sheet ID matches.

## Recommendations
- Store API keys in n8n credentials (do not hardcode in the JSON). Replace inline `YOUR-API-KEY-HERE` with a credential reference.
- Add rate-limiting and error retries around `Search Internet` and LLM calls for robustness.
- Consider caching top search results or using a vector DB to avoid repeated identical searches.

---
Workflow file: Research_and_Content_Creation_Agents.json
