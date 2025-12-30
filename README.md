
# n8n Agents Collection

A curated collection of ready-to-import n8n workflows (JSON exports) implementing agents and automations you can adapt for your own n8n instance. Workflows cover common automation patterns like email/calendar agents, content generation, voice/TTS pipelines, RAG/embeddings, lead capture, and human-in-the-loop flows.

Why this repo
- Save time: reuse tested workflow patterns rather than building from scratch.
- Learn: see example prompts, output parsers, and tool integrations used in production-like workflows.
- Customize: each JSON is intended as a starting point — edit nodes, prompts, and credentials to match your environment.

Quick start
1. Install n8n (self-hosted or cloud): https://n8n.io/
2. In the n8n editor choose **Import** and select a workflow JSON from this repo.
3. Configure credentials referenced by nodes (OpenAI/Anthropic/Google/Gmail/Drive/ElevenLabs/Pinecone/etc.).
4. Test the workflow using the editor's Execute/Trigger tools before activating.

Best practices
- Replace placeholder API keys and temporary upload services with production-ready secrets and storage (e.g., S3/R2).
- Keep credentials in n8n's credential store — never commit secrets to the repo.
- Add human-approval steps for high-impact actions (sending emails, modifying calendars, publishing content).

Contributing
- Add new exported workflow JSON files or improved READMEs per folder.
- Use clear file names and include a short README for non-trivial workflows.
- Open issues for bugs, suggestions, or requests.

License & contact
- No license included. Add a `LICENSE` file if you plan to publish or share these workflows publicly.
- For help adapting any workflow, open an issue or add a README note in the relevant folder.

Enjoy building with n8n — import workflows, adapt prompts, and iterate quickly.

