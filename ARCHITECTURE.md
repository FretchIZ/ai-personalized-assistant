# AI Personalized Assistant — Architecture

This document describes the high-level architecture and runtime workflow for the ai-personalized-assistant project.

Goals
- Provide a personalized AI assistant with short- and long-term memory, integrations, and action execution.
- Use RAG (Retrieval-Augmented Generation) to ground responses in user data and documents.

Components
1. Frontend (web/mobile)
   - Next.js / React app providing chat UI, conversation history, settings, profile management.
   - Responsible for authentication, streaming responses, and presenting actions.

2. Backend API
   - Node.js (NestJS/Express) or Python (FastAPI) that exposes REST/GraphQL endpoints for conversations, user management, integrations, and document ingestion.
   - Handles auth, rate limits, request validation, and orchestration.

3. Worker / Orchestrator
   - Background workers (BullMQ / Sidekiq / RQ) that perform heavy tasks: embedding computation, document ingestion, third-party action execution, scheduled jobs.

4. LLM Provider / Model Layer
   - External LLMs (OpenAI, Anthropic) or self-hosted models.
   - Uses prompt templating and tool routing. Track token usage per request.

5. Vector DB (memory store)
   - Qdrant / Pinecone / Weaviate / Milvus / FAISS for semantic retrieval of embeddings.

6. Database & Object Store
   - Postgres for structured data (users, conversations, messages, metadata).
   - S3-compatible bucket for attachments and documents.

7. Integrations
   - OAuth connectors and adapter services for Gmail, Calendar, Slack, Notion, etc.

8. Observability
   - Metrics (Prometheus/Grafana), centralized logs (ELK), and error monitoring (Sentry).

Runtime Conversation Workflow
1. Client submits user message to backend (/api/v1/converse).
2. Backend authenticates and enforces rate limits and quotas.
3. Conversation service loads:
   - User profile & preferences from Postgres
   - Short-term memory: recent conversation turns
   - Long-term memory: top-K retrieved documents/memories from the vector DB based on an embedding of the message
4. Prompt assembly: system prompt + user profile + retrieved memories + recent turns + current user message
5. Optional intent classification / tool routing step (fast model or rule-based)
6. LLM call with assembled prompt; stream tokens if enabled
7. Post-process LLM response, detect structured actions (JSON) and enqueue tasks to worker if third-party API calls are needed
8. Store message and computed embeddings for memory; update conversation metadata
9. Return response to client and log telemetry (latency, token counts, errors)

Security & Privacy (summary)
- TLS for all transport
- Encrypt sensitive fields at rest (e.g., OAuth tokens, PII)
- Per-user access controls and isolation
- User controls for memory: view, export, delete
- Stored secrets in vault / platform secrets manager

Repository layout (recommended)
/
├─ README.md
├─ ARCHITECTURE.md
├─ docs/
│  └─ design.md
├─ web/
├─ server/
├─ workers/
├─ infra/
└─ .github/workflows/
