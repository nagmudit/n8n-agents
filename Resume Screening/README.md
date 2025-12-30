# Resume Screening

Automated resume intake and screening workflow that ingests resumes (Gmail), standardizes file types, extracts text, evaluates candidates against a job description with the `Recruiter Agent`, and appends structured screening results to a Google Sheet.

## Purpose
- Automate candidate intake and initial screening: accept resumes via Gmail, save originals to Google Drive, extract text from DOC/PDF/TXT, evaluate against a job description using the `Recruiter Agent`, extract contact info, and append a structured report to the `Resume Screener` Google Sheet.

## Key Nodes
- `Receive Resume` — Gmail trigger that downloads attachments.
- `File Type` — switch node to route DOCX / PDF / TXT flows.
- `Upload to Drive` — stores original resume in Drive (folder ID configured in node).
- `Convert Word to Docs` + `Get Doc` — convert DOCX to Google Docs and export as PDF when needed.
- `Extract from File` / `Extract from File1/2/3` — extract text from PDF/DOC/TXT.
- `Set Resume` — normalizes extracted text into the `resume` field.
- `Get Job Description` — fetches the job description from Drive (documentId configured) and extracts its text.
- `Recruiter Agent` — LLM agent that compares candidate resume to job description and outputs strengths, weaknesses, risk/reward, fit rating (structured output expected).
- `Structured Output Parser` — enforces the JSON schema for the agent output.
- `Information Extractor` — extracts `First Name`, `Last Name`, and `Email Address` from the resume.
- `Append Data` — appends a row to the `Resume Screener` Google Sheet with parsed fields and the agent’s report.

## Credentials & Setup
- `Gmail OAuth2` — credential for the `Receive Resume` trigger.
- `Google Drive OAuth2` — used by `Upload to Drive`, `Get Doc`, `Get PDF`, and `Get TXT` nodes (ensure Drive folder ID is accessible).
- `OpenAI` (or chosen LM) — configured for `Recruiter Agent`, `o4-mini`, and other LLM nodes.
- `Google Sheets OAuth2` — credential for `Append Data` (sheet ID is referenced in node).

Suggested env vars / credential names:
- `GMAIL_CREDS`, `GOOGLE_DRIVE_CREDS`, `OPENAI_API_KEY`, `GOOGLE_SHEETS_CREDS`.

## How It Works
1. Mail with attached resume arrives → `Receive Resume` triggers and downloads attachments.
2. `File Type` switch routes attachments to the appropriate conversion/extraction nodes (DOCX → convert → export PDF → extract; PDF → extract; TXT → extract text).
3. Extracted text is set to `resume` and the job description is pulled via `Get Job Description` then extracted.
4. `Recruiter Agent` analyzes `resume` against the job description and returns a structured screening report (strengths, weaknesses, risk/reward, overall fit and justification).
5. `Information Extractor` pulls contact fields from resume text.
6. `Append Data` writes results and metadata (resume Drive link, scores) into the `Resume Screener` Google Sheet.

## Usage & Testing
- Test by sending an email with an attached resume to the Gmail account configured in the `Receive Resume` node.
- Verify `Upload to Drive` stores the original file; confirm `Extract from File` returns readable text.
- Confirm the `Recruiter Agent` node returns all fields required by the `Structured Output Parser` (candidate_strengths, candidate_weaknesses, risk_factor, reward_factor, overall_fit_rating, justification_for_rating).
- Check the target Google Sheet to ensure a new row is added with the parsed outputs.

## Output Schema (enforced)
- The `Structured Output Parser` expects an object with:
  - `candidate_strengths` (array of strings)
  - `candidate_weaknesses` (array of strings)
  - `risk_factor`: { `score`: Low|Medium|High, `explanation`: string }
  - `reward_factor`: { `score`: Low|Medium|High, `explanation`: string }
  - `overall_fit_rating` (integer 0–10)
  - `justification_for_rating` (string)

## Troubleshooting
- Missing text from `Extract from File`: confirm file is not image-only; if scanned images, use OCR or improve input quality.
- Agent output schema mismatch: ensure `Recruiter Agent` prompt and `Structured Output Parser` schema align; test with sample resume/job description pairs.
- Drive/Sheets permissions errors: ensure OAuth2 credentials have `drive.file` and `sheets` scopes and the sheet/folder IDs are correct.
- Wrong field extraction: update regexes or the `Information Extractor` attributes if resumes use different layouts.

## Privacy & Compliance
- This workflow handles PII (names, emails). Run on private infrastructure, restrict access to n8n, and secure Google Drive/Sheets with appropriate ACLs. Consider redaction/encryption for storage.

---
Sample files: `Brian Chen.txt`, `Job Title_ AI Solutions Architect.txt`, workflow: `Resume_Screening.json`
