# Sales Team — Human In The Loop (HITL)

Automated sales workflow where an AI `Sales Agent` drafts outreach emails for incoming leads and a human reviewer provides feedback/approval before messages are sent. Includes revision loop and feedback classification.

## Purpose
- Automate initial lead outreach while keeping final approval with a human: generate concise, persuasive emails from lead fields, reference prior projects from Airtable, request human approval, accept feedback, revise via an AI `Revision Agent`, and send the approved email.

## Key Nodes
- `Airtable Trigger` — triggers on new lead rows (field: `Created`).
- `Project Database` — Airtable lookup tool used by the agent to find past similar projects.
- `Sales Agent` — LangChain-style agent that composes an initial subject + email body from lead fields and project examples. Signs off as "Jim, Dunder AI".
- `Structured Output Parser` — enforces the `subject` and `email` fields returned by the agent.
- `Set Email` — maps agent output into a workflow variable.
- `Get Feedback` — sends the draft to a human (via `gmail` node configured with `sendAndWait`) and collects free-text feedback.
- `Check Feedback` — text classifier that categorizes human feedback into `Approved` or `Declined`.
- `Revision Agent` — revises the draft based on feedback and re-runs output parsing.
- `Send Email` — final send (Gmail) when feedback classification is `Approved`.

## Credentials & Setup
- `Airtable` token — for `Airtable Trigger` and `Project Database`.
- `Anthropic`, `OpenAI`, or `Google PaLM` credentials — for the `Sales Agent`, `Revision Agent`, and `textClassifier` language models.
- `Gmail OAuth2` — used by `Get Feedback` (sendAndWait) and `Send Email` nodes. Ensure `sendAndWait` behavior is acceptable for your mailbox.

Suggested env vars / credential names:
- `AIRTABLE_API_KEY`, `ANTHROPIC_KEY` (or `OPENAI_KEY`/`GOOGLE_PALM_KEY`), `GMAIL_CREDS`.

## How the Flow Works
1. New lead record in Airtable → `Airtable Trigger` fires.
2. `Sales Agent` composes subject + email using lead fields and a similar past project from `Project Database`.
3. `Structured Output Parser` validates `subject` and `email` structure; `Set Email` saves the draft.
4. `Get Feedback` emails the draft to the human reviewer using `sendAndWait` and captures free-text feedback.
5. `Check Feedback` classifies feedback as `Approved` or `Declined`.
6. If `Approved` → `Send Email` sends the message to the lead. If `Declined` → `Revision Agent` rewrites the email per feedback and the loop continues until `Approved`.

## Testing
- Populate Airtable with a test lead row containing `name`, `email`, `intent`, `budget`, `companyName`, `projectDescription`, and `timeline`.
- Trigger the workflow and verify the `Sales Agent` output appears in `Set Email`.
- Validate `Get Feedback` sends the draft to your reviewer address; reply with example feedback such as "Looks good" or "Please remove pricing details" to test classifier branches.
- Confirm `Revision Agent` updates the email text and that `Send Email` only fires after `Approved` classification.

## Troubleshooting
- Agent returns invalid structure: open the `Sales Agent` node and ensure its prompt + `Structured Output Parser` schema match (subject + email required).
- `Get Feedback` not receiving replies: `sendAndWait` relies on Gmail; ensure credentials and webhook IDs are set and that the mailbox can receive replies.
- Classifier mislabels feedback: add more examples to the `Check Feedback` categories or switch to a stronger classifier model.
- Infinite loops: add a max revision counter or require manual override after N revisions.

## Recommendations
- Use a shared reviewer inbox address and limit reviewers to a small set of trusted people.
- Log approvals and feedback in Airtable for auditability.
- For production, replace `sendAndWait` with a webhook-based human-review UI to avoid mailbox polling and rate limits.

---
Workflow file: Sales_Team__HITL.json
