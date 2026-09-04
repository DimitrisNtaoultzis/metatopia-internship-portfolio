# Software Engineering Internship — Metatopia (Zeus Platform)

*Summer 2026 (July–August), Backend / AI-RAG track*

This repo is a **portfolio summary** of the work I did during my internship at [Metatopia](https://metatopia.gr), on their **Zeus Platform** — a multi-tenant hospitality SaaS product (guest app, staff dashboard, and an AI concierge chatbot) built with **Spring Boot 3.5 / Java 21** on the backend, **Next.js/TypeScript** for the admin dashboard, and **React Native/Expo** for the guest mobile app, backed by **PostgreSQL + pgvector** for RAG/semantic search.

The actual source code belongs to Metatopia and lives in private company repositories, so it isn't included here. This is a description of the problems I worked on, the decisions involved, and what I learned — not a code dump.

## What I worked on

### 1. Backend feature development
- **Transactional email notifications** — built the core SMTP email-sending service from scratch and wired it into the approval workflow, so staff get notified by email when a new item lands in the approval queue.
- **File upload integration** — designed and implemented how the platform's generic file-upload API gets wired into content fields across the product (venue logo/banner, accommodation gallery images, activities, offers, food & drink venues, emergency contacts, hotel services). This involved real design trade-offs I worked through with a senior engineer: public vs. authenticated read access for images, how upload responses are correlated with the fields that reference them, and cleanup of orphaned files on replacement — all while keeping backward compatibility with pre-existing external image URLs and requiring no data migration.
- **Bug fixing** — found and fixed a Hibernate/ORM caching bug where an entity's response could show stale data right after being created/updated in the same transaction, by tracing it to a JPA identity-map/session caching issue and applying a targeted fix.
- Every change shipped with unit/integration tests and went through the team's merge-request review process on GitLab.

### 2. PDF-to-knowledge-base ingestion pipeline (AI/RAG)
Proposed, scoped, and helped implement a feature to let the hotel's knowledge base ingest content directly from PDF documents (policies, price lists, local guides) instead of manual entry only:
- Investigated feasibility, identified architectural risks (e.g. resource contention between a bulk import job and the live guest chatbot sharing the same inference backend), and wrote up a phased implementation plan that was reviewed and approved by my supervisor.
- Implemented the **PDF text-extraction service** and the **agentic structuring service** (using a local LLM to turn raw extracted text into structured, categorized knowledge-base entries), including handling long documents, malformed files, and content-length limits.
- Ran a systematic model comparison during implementation and found meaningful quality differences between candidate local models for this extraction task, which fed into the team's model-choice decisions.

### 3. AI chatbot quality assurance & root-cause debugging
This was the largest and most technically interesting part of the internship: a multi-week effort to evaluate and improve the reliability of the guest-facing AI concierge chatbot (a RAG pipeline over the knowledge base, running on local LLMs via Ollama).

- **Systematic QA testing** — designed and ran structured test scenarios across multiple guest personas (including non-English-speaking guests) and multiple candidate LLMs, verifying every claim the bot made against the actual backend state (database rows, not just chat transcripts). This surfaced a recurring, high-impact pattern: the bot would confidently tell guests an action had been completed (a booking, a maintenance request, a refund) when nothing had actually happened server-side.
- **Root-cause analysis** — went from "the bot hallucinates" as a symptom to specific causes in the prompt-assembly and orchestration code: retrieved knowledge-base content wasn't actually being passed to the model (only titles were), and the system prompt itself told the model to confirm actions unconditionally with no check against real backend state. Distinguished orchestration/prompt bugs from retrieval bugs and from model-capacity issues by testing across three different local models and showing the same failure pattern persisted regardless of model size or family — evidence the root cause was architectural, not "just use a bigger model."
- **Fix implementation** — implemented and shipped the fix that threads the *real* outcome of a guest request (task created / approval queued / nothing happened) into the prompt, so the model is given a true signal to confirm or not confirm, rather than an unconditional instruction to always confirm.
- **Verification & further findings** — after my fix and a teammate's related fix landed, I re-ran the full test suite against live state to verify the fixes actually worked end-to-end, which surfaced further, more specific findings: gaps in conversation-history awareness for retrieval, a missing task-update/cancellation mechanism (the system could create tasks but never modify or cancel them — including in negation cases like "cancel that" being misread as a *new* request), and a staff-notification gap on chat escalation. Consolidated all findings into a de-duplicated report with root causes and suggested fixes for the team.

### 4. Process improvement proposals
- Investigated the state of CI/CD and static analysis across the product's repositories (none existed at the time) and wrote a proposal for introducing a first CI pipeline paired with SonarQube static analysis, including a phased rollout plan and a self-hosted-vs-cloud comparison.
- Wrote up two follow-on feature proposals for the chatbot (fixing the staff-paging gap on escalation, and a staff-facing analytics dashboard for conversation/escalation metrics) based on findings from the QA work above.

## Skills exercised

`Java` · `Spring Boot` · `PostgreSQL` · `pgvector` · `Hibernate/JPA` · `RAG pipelines` · `LLM prompt engineering & evaluation` · `Ollama (local LLM inference)` · `Git/GitLab merge-request workflows` · `unit & integration testing` · `technical writing & root-cause reporting`
