# Phase 1 Architecture & Delivery Plan — WhatsApp AI Assistant MVP

This document frames the Phase 1 MVP: a pragmatic WhatsApp → AI → backend flow with basic admin controls. It is intentionally lean—enough structure to validate value quickly, without over-engineering. Backend is standardized on **Node.js with Express** for delivery speed and ecosystem fit.

## 1) Goals & Scope (Phase 1 / MVP)
- Prove the end-to-end path: WhatsApp inbound → AI understanding → ticket logic → admin visibility.
- Keep the backend simple (Node.js with Express) while integrating the WhatsApp Business Cloud API.
- Use an LLM (OpenAI or similar) for language detection, intent classification, and structured data extraction.
- Persist conversations and tickets, including emergency-handling rules, and expose a minimal admin surface for listing, detail, and exports.
- Deliver code, demo videos per milestone, and a checklist-driven validation.

## 2) Architecture Overview
The system uses WhatsApp as the single user channel, a lightweight Node/Express backend to orchestrate AI calls and ticket logic, and PostgreSQL to anchor state. An admin UI consumes the same APIs used by the core flow.

- **Client channel:** WhatsApp Business Cloud API for inbound webhooks and outbound sends.
- **Ingress & routing:** HTTPS webhook endpoint that verifies signatures and normalizes messages.
- **LLM services:** OpenAI (or equivalent) via a thin gateway for language detect, classify, and extract.
- **Backend/API:** REST (or lean GraphQL) service on Express managing sessions, orchestration, ticket lifecycle, and admin endpoints.
- **Database:** PostgreSQL for conversations, tickets, users/admins, and audit events.
- **Admin UI:** Minimal web UI for tickets (list/detail), basic actions, and exports.
- **Observability:** Basic structured logs and ticket/message audit trails.

### Text Diagram (logical view)
```
WhatsApp User
    │
    ▼
WhatsApp Business Cloud API (webhook → verify → normalize)
    │
    ▼
Backend Service (Node.js/Express)
    ├─ Ingress Controller (auth, signature verify, rate limits)
    ├─ Message Orchestrator
    │    ├─ LLM Gateway (OpenAI or equivalent)
    │    └─ Rules/Policy (language detect, classify, extract)
    ├─ Ticket Engine (create/update/escalate; emergency flagging)
    ├─ Persistence Layer (DB ORM/queries)
    └─ Admin API
         └─ Admin UI (tickets list/detail, actions, exports)
    │
    ▼
Database (conversations, tickets, audit, admin users)
```

## 3) Data Flow (happy path)
1) WhatsApp sends a message to the webhook; the backend verifies the signature and accepts the event.  
2) The payload is normalized (user, message, media refs) and logged for audit.  
3) The orchestrator calls the LLM gateway to detect language, classify intent, and extract structured fields (e.g., issue type, urgency).  
4) The ticket engine decides whether to create or update a ticket, flags emergencies, and attaches transcript and metadata.  
5) A templated reply (aligned to intent/language) is sent back via WhatsApp.  
6) Admin UI reads the same records via API for list/detail, actions, and exports; actions feed back to the ticket engine.  

## 4) Key Components & Responsibilities
- **Webhook Ingress:** Signature verification, retry handling, idempotency on message IDs, and payload normalization.
- **Message Orchestrator:** Routes to LLM gateway, applies guardrails/timeouts, and falls back to simple rules if LLM is down.
- **LLM Gateway:** Provider abstraction for model calls; caps lengths, handles refusals, and enforces output schema.
- **Ticket Engine:** Manages ticket lifecycle, deduplication, emergency/escalation rules, and audit logging.
- **Persistence:** Relational schema for users, conversations, messages, tickets, events, and admins with indexes for fast lookups.
- **Admin API + UI:** Basic auth, ticket list/detail, status changes, notes, and CSV/JSON exports.
- **Observability:** Correlation IDs in logs and a simple health endpoint.

## 5) Technology Choices (pragmatic defaults)
- **Backend:** Node.js with Express (chosen for speed and ecosystem).
- **Database:** PostgreSQL with migrations via Prisma or Knex.
- **LLM:** OpenAI GPT-4/GPT-4o or similar, behind an abstraction to allow provider swaps.
- **Admin UI:** Lightweight React/Vite or server-rendered pages; minimal components to satisfy the checklist.
- **Infra:** Single container/VM for MVP; HTTPS terminated via managed cert; env-var based secrets.

## 6) Milestones, Deliverables, and Demos (fixed scope)
**Milestone 1 — Ingress & Core Flow**  
- WhatsApp webhook verified; outbound send works in sandbox.  
- Message normalization, logging, and idempotency in place.  
- Simple hello-flow responding over WhatsApp.  
- Demo video + validation checklist.

**Milestone 2 — LLM Integration & Ticket Engine**  
- LLM gateway for language detect, classification, and structured extraction.  
- Ticket creation/update with emergency flag logic; schema migrated.  
- Functional WhatsApp → AI → ticket flow with sample intents.  
- Demo video + validation checklist.

**Milestone 3 — Admin Panel & Exports**  
- Admin auth (basic), ticket list/detail, status updates, notes, exports (CSV/JSON).  
- Ticket audit trail visible.  
- Demo video + validation checklist.

## 7) Validation Checklist (per milestone)
- WhatsApp webhook verified; messages round-trip with expected text.  
- Idempotent handling on duplicate webhook deliveries.  
- LLM returns correct language, intent, and extracted fields for test set.  
- Tickets created/updated with expected statuses and emergency rules.  
- Admin UI shows list/detail; allows status change and exports.  
- Logs carry correlation IDs; health endpoint returns OK.  

## 8) Assumptions & Constraints
- Phase 1 targets sandbox/limited-scale traffic in a single region.  
- Validation is behavior-based (demo + checklist), not source review.  
- Scope is fixed for this phase; changes queue to later phases.  
- WhatsApp sandbox access, LLM API keys, and DB credentials are provided.  

## 9) Risks & Mitigations
- **LLM latency/availability:** Timeouts with cached or rule-based fallbacks.  
- **Webhook retries/duplication:** Idempotency keys per message and safe upserts.  
- **Data quality in extraction:** JSON-shaped prompts with validation before use.  
- **Overbuild risk:** Keep schema/UI minimal; defer extras to later phases.  
- **Security basics:** HTTPS, signature verification, minimal admin RBAC, secrets in env vars.  

## 10) Handover Artifacts
- Repository with README/setup, sample env, and migration scripts.  
- Postman/HTTPie collection for webhook replay and admin APIs.  
- Demo videos per milestone and a final walkthrough.  
- Runbooks for deploy, key rotation, restarts, and log locations.  
