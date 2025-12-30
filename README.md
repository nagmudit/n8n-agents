
# n8n Agents Collection

This repository is a collection of n8n workflow agents and example automations designed to be imported into an n8n instance. Each file in the root and subfolders is an exported n8n workflow (JSON) that implements an automation or "agent" for common tasks such as customer support, voice agents, email automation, content creation, and lead generation.

Key features
- A variety of ready-to-import n8n workflows (.json) covering: customer support voice agents, ElevenLabs voice integrations, Gmail autoreply and scheduling automations, LinkedIn automation, lead generation, and script/video generation.
- A `JARVIS/` subfolder containing specialized agent workflows and supporting examples.
- Small, focused workflows intended to be adapted and extended for your own n8n instance.

Repository layout (high-level)
- Root: many standalone exported workflows (JSON files) — import any into n8n using the import workflow feature.
- JARVIS/: a collection of related agents and a README with more context.
- "Script and Video Generation/": additional workflows and notes for media generation tasks.

How to use
1. Install n8n (self-hosted or cloud): https://n8n.io/
2. In n8n editor, choose "Import" and select one of the JSON files from this repo.
3. Inspect credentials and node configuration — replace any placeholder API keys or webhook URLs with your own.
4. Activate or test the workflow in n8n.

Tips
- Review each workflow's credential nodes (e.g., ElevenLabs, Gmail, OpenAI) and configure them in your n8n instance before activating.
- Use the workflow versions in this repo as starting points — adapt node logic, filters, and triggers to match your environment.

Contributing
- Add new exported n8n workflows or improvements to existing ones as separate JSON files.
- Name conventions: use descriptive file names (e.g., `Customer_Support_Voice_Agent.json`).
- Open an issue to discuss larger changes or share usage notes.

License & contact
- This repository does not include a license file. If you plan to reuse or share these workflows, consider adding a `LICENSE` file to clarify terms.
- For questions or help adapting workflows, open an issue in this repo or add a README note in the relevant folder.

Enjoy building with n8n — import the workflows and customize them to automate tasks quickly.

