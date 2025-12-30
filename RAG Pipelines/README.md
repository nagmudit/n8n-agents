# RAG Pipelines

Overview
- Purpose: Ingest Google Drive documents into a Supabase vector store and provide a chat interface that uses retrieval-augmented generation (RAG) to answer queries.
- Location: `RAG Pipelines/RAG Pipelines.json` (n8n workflow export)

Pipelines
- Pipeline 1 — Upload (fileCreated):
  - `Google Drive Trigger` → `Download file` (converts Docs→PDF) → `Default Data Loader` → `Embeddings OpenAI` → `Supabase Vector Store` (insert into `documents`).
- Pipeline 2 — Update (fileUpdated):
  - `Google Drive Trigger1` → `Delete a row` (remove old vectors matching fileName) → `Download file1` → `Default Data Loader1` → `Embeddings OpenAI2` → `Supabase Vector Store2` (insert updated vectors).
- Pipeline 3 — Delete (recycling bin):
  - `Google Drive Trigger2` (Recycling Bin watch) → `Delete a row1` (remove vectors for moved/deleted files).

Query / Chat Flow
- `When chat message received` (chat webhook) → `AI Agent` using `OpenRouter Chat Model`.
- `Supabase Vector Store1` is configured as a retrieval tool (`mode: retrieve-as-tool`) to supply relevant document chunks for context.

Key Nodes & Config
- Google Drive triggers + `Download file` nodes (with `docsToFormat: application/pdf` conversion).
- `documentDefaultDataLoader` nodes attach metadata (fileName, date).
- Embeddings: `Embeddings OpenAI*` nodes (OpenAI API credential required).
- Vector DB: `vectorStoreSupabase` nodes using `documents` table (Supabase credentials required).
- Chat model: `OpenRouter Chat Model` mapped to `AI Agent`.

Required Credentials
- Google Drive OAuth2 (Drive triggers & download).
- Supabase API (vector insert/retrieve/delete).
- OpenAI API (embeddings) and OpenRouter (chat model).

Testing
1. Configure credentials and set the Drive folder IDs to your own.
2. Drop a document into the watched folder — confirm a new record appears in Supabase `documents`.
3. Update the file — confirm the old vectors are removed and new ones inserted.
4. Move the file to the recycling bin folder — confirm corresponding vectors are deleted.
5. Send a message to the chat webhook and verify `AI Agent` returns answers using retrieved docs.

Notes
- The workflow is currently `active: false` — enable before testing.
- Adjust folder IDs, table name (`documents`), and model mapping to match your environment.
- Sticky notes in the workflow provide a setup guide and pipeline descriptions.

Files
- Workflow: [RAG Pipelines/RAG Pipelines.json](RAG%20Pipelines/RAG%20Pipelines.json)

Last updated: 2025-12-31
