# Decision: MinhaIncorporadora content ingestion & knowledge backend

## Status

Accepted — 2026-06-25

## Context

MinhaIncorporadora must answer brokers from a developer's own projects, files and
commercial data (manuals, floor plans, images, videos, price tables, payment terms,
availability). The robot has its own database but no file repository / knowledge
structure, and each developer (EBM is the reference) stores content differently. See
`rfcs/RFC-0005-minhaincorporadora-content-ingestion-backend.md`.

## Decision

Build the content backend as an evolution of the existing plugin, with these locked
choices:

1. **Storage in-container, on the persistent volume.** Originals are kept under
   `/opt/data/files/...`, which is a **host bind mount** (`${HERMES_DATA_DIR}:/opt/data`),
   so they **survive container recreation and any redeploy** (the deploy git-resets
   only `product-src/`, never `files/`). `storage_key` is stored relative to the
   storage root so references survive a host/path change. External storage (S3 /
   Supabase Storage) is deferred behind the same `StorageBackend` abstraction. Ops
   backups must cover `$DATA_DIR/files` and the database together.
2. **AI via OpenRouter, per-format specialization.** A `ModelRouter` selects the best
   model per media type and task (images/floor plans → vision/OCR; pdf/doc/ppt →
   native parse + text model; xlsx/csv → native parse; video → ASR + keyframe
   vision). Native parsers run first; AI handles vision, extraction, summarization and
   ambiguous classification. DeepSeek (or similar) is an economical text option.
   Secret `OPENROUTER_API_KEY` via `hermes-infra`.
3. **Search: Postgres FTS now, pgvector later.**
4. **Commercial grain:** payment terms belong to the project (`condicoes_pagamento`
   per `empreendimento`); units belong to the project; **availability is a property of
   the unit** (`unidades.disponibilidade`).
5. **Tenancy is physical.** One container = one incorporadora = one bot
   (WhatsApp/Telegram unique per container). No multi-tenant routing; the developer is
   a singleton row; `MODE=dedicated` is the production norm.

Hard facts extracted by AI are provisional and require human review before becoming
canonical, preserving the zero-invention rule (RFC-0003 / AGENTS).

## Consequences

- `hermes-minhaincorporadora`: new migrations `0003`–`0006`; `ingest/` package
  (storage, sources, classify, extract, pipeline); `services/` additions; `ai.py`
  AI seam; new admin functions and agent tools. Delivered in phases (RFC-0005).
- `hermes-infra`: a new secret `OPENROUTER_API_KEY` per developer/profile; the
  in-container file directory is part of the profile's data dir (already mounted).
- Implementation starts with Phase 1 (no real AI; deterministic stubs) so the
  pipeline and tests run without network/keys.

## Related Repositories

- `hermes-minhaincorporadora` — the backend.
- `hermes-infra` — `OPENROUTER_API_KEY` secret; data-dir storage.
- `hermes-roadmap` — `rfcs/RFC-0005-minhaincorporadora-content-ingestion-backend.md`.
