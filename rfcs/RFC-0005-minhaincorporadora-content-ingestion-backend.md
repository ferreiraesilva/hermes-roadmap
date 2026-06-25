# RFC-0005: MinhaIncorporadora Content Ingestion & Knowledge Backend

## Status

Accepted (design) — phased; Phase 1 ready to implement on `hermes-minhaincorporadora`

## Product

MinhaIncorporadora

## Summary

MinhaIncorporadora answers brokers over WhatsApp using a developer's
(incorporadora's) own projects, files and commercial data. Today the plugin only
serves a thin, hand-seeded model (`empreendimentos`, `unidades`, `midias`). This RFC
designs the backend that **ingests, classifies, structures and exposes** a
developer's content — project descriptions, broker manuals, floor plans, images,
videos, price tables, payment terms and availability — so the assistant
(see RFC-0003: Assistente do Gerente de Vendas) can answer questions and deliver the
right original file.

This is an **evolution of the existing plugin**, not a greenfield: same schema
`minhaincorp`, same conventions (deterministic Python + typed tools + Postgres as
source of truth + zero invention).

EBM is the reference developer; its public repository is the custom JS app
"FileHub EBM" (`arquivos.ebm.com.br`, navigated via `?path=`), so ingestion needs a
per-developer **source adapter** rather than static scraping.

## Motivation

Each developer stores content differently (folder layouts, file formats, naming).
The robot already has its own database but **no file repository / knowledge-file
structure**. We need a reliable way for each robot/developer to store, classify,
search and retrieve files and the data extracted from them — instead of forcing the
assistant to re-read raw files on every question, and without ever inventing facts.

## Locked decisions (this session)

1. **Storage is in-container.** Few GB / few files; keep originals on the container
   filesystem (`/opt/data/files/...`); back up via ops. Storage stays behind an
   abstraction for a future external backend, but no S3/Supabase Storage now.
2. **AI = per-format specialization via OpenRouter.** A model router picks the best
   model per media type and task. Native parsers run first (cheap, grounded); AI
   handles vision, extraction, summarization and ambiguous classification. DeepSeek
   (or similar) is an economical text option, not merely a fallback.
3. **Search: Postgres FTS now, pgvector later.** Both phases are recorded here.
4. **Commercial grain:** payment terms belong to the **project**; units belong to
   the **project**; **availability is a property of the unit**.
5. **Tenancy is physical:** one container = one incorporadora = one bot
   (WhatsApp/Telegram unique per container). No multi-tenant routing; the developer
   is effectively a singleton row. `MODE=dedicated` is the production norm.

## Domain model

```text
incorporadora (developer = the container = the bot; singleton)
└── regiao (praça: city / region / market)                 [NEW]
    └── empreendimento (project)            [+ regiao_id, + source_path]
        ├── unidade (price + availability)        availability lives here
        ├── condicao_pagamento (per project)                [NEW / formalized]
        ├── arquivo (file repository)                       [NEW — core]
        │     └── extracao (knowledge: summary + structured data)  [NEW]
        └── gerentes / imobiliarias / corretores (existing, RFC-0003)
lote_importacao (JSON import / crawl batch, provenance)     [NEW]
arquivo_evento (append-only pipeline log)                   [NEW]
```

`incorporadora_id` is kept on new tables for continuity with the existing schema but
is a constant (resolved once); no tenant-scoping machinery is built.

## Schema (additive, idempotent migrations in schema `minhaincorp`)

- **`0003_regioes.sql`** — `regioes(id, incorporadora_id, nome, slug, uf, tipo,
  descricao)`; `empreendimentos += regiao_id, source_path`.
- **`0004_arquivos.sql`** — file repository:
  - enums `arquivo_categoria` = {`manual_corretor`, `material_descritivo`, `planta`,
    `imagem_render`, `video`, `tabela_preco`, `condicao_pagamento`, `disponibilidade`,
    `documento_juridico_comercial`, `desconhecido_revisar`}; `ingestao_status` =
    {`descoberto`, `baixado`, `armazenado`, `classificado`, `extraido`, `pronto`,
    `falhou`}; `review_status` = {`nao_requer`, `pendente`, `aprovado`, `rejeitado`}.
  - `arquivos(id, incorporadora_id, regiao_id?, empreendimento_id?, unidade_id?,
    categoria, titulo, descricao, source_url, source_path, origem, storage_backend,
    storage_key, mime_type, ext, tamanho_bytes, hash_sha256, status_ingestao,
    classificacao_confianca, classificado_por, review_status, review_motivo,
    criado_em, atualizado_em)`; `UNIQUE(incorporadora_id, hash_sha256)` for dedup;
    indexes on `(empreendimento_id, categoria)` and `(incorporadora_id, review_status)`.
- **`0005_extracoes.sql`** — `extracoes(id, arquivo_id, incorporadora_id,
  empreendimento_id?, tipo, resumo, dados_estruturados jsonb, modelo_ia,
  prompt_versao, confianca, review_status, busca_tsv tsvector)`; GIN index on
  `busca_tsv` (FTS v1; pgvector embedding column added in Phase 3).
- **`0006_importacao_revisao.sql`** — `condicoes_pagamento(id, empreendimento_id,
  nome, texto, estruturado jsonb, origem, review_status)`; `lotes_importacao(...)`;
  `arquivo_eventos(arquivo_id, de_status, para_status, detalhe, criado_em)`.

**Zero-invention guard:** AI-extracted hard facts (price/availability) land in a
**provisional** layer (`extracoes` + `review_status='pendente'`). They become canonical
(`unidades` / `condicoes_pagamento`) only after **human approval**. The assistant never
serves unreviewed extracted facts as certain.

## Standard JSON import format (versioned)

```jsonc
{
  "schema_version": "1.0",
  "developer": { "slug": "ebm", "nome": "EBM" },
  "regions": [ { "slug": "goiania-rmg", "nome": "Goiânia e Região Metropolitana",
                 "uf": "GO", "tipo": "regiao" } ],
  "projects": [ {
    "slug": "rota-panorama", "nome": "Rota Panorama", "region": "goiania-rmg",
    "status": "lancamento", "cidade": "Goiânia",
    "source_path": "Goiânia e Região Metropolitana/Rota Panorama",
    "payment_terms": [ { "nome": "Padrão", "texto": "Entrada 20% + ...",
                         "estruturado": { "entrada_pct": 20 } } ],
    "units": [ { "numero": "101", "tipologia": "2 quartos", "area_m2": 58,
                 "preco": 320000, "disponibilidade": "disponivel" } ],
    "files": [ { "source_url": "https://arquivos.ebm.com.br/.../manual.pdf",
                 "titulo": "Manual do corretor", "categoria": "manual_corretor" } ]
  } ]
}
```

Import = idempotent upsert by `(developer, region/project slug)`. Referenced files
enter the ingestion pipeline. `categoria` is optional — if omitted, the file is
classified. `importar_json()` validates `schema_version` and returns a batch summary.

## Storage architecture

- **`StorageBackend`** (`put/get/url/exists`) — default **local filesystem** under
  `/opt/data/files/<incorporadora>/<empreendimento>/...`. Keeps an owned copy of each
  original (stable for WhatsApp delivery) alongside the recorded `source_url`. Config:
  `MINHAINCORP_STORAGE_BACKEND` (default `localfs`), `MINHAINCORP_STORAGE_ROOT`.
  External backends (Supabase Storage / S3) are a future implementation of the same
  interface.
- **`SourceAdapter`** (`list(path)`, `fetch(entry)`) — accommodates each developer's
  different folder structure without touching the rest. Implementations:
  `HttpFolderAdapter` (generic), **`FileHubAdapter`** (EBM — `# TODO`: consume the
  FileHub JSON API behind `?path=`), `LocalFolderAdapter`.

## Ingestion pipeline (state machine, idempotent)

```text
descoberto → baixado → armazenado → classificado → extraido → pronto
                                          └─(low confidence)→ review_status=pendente
any step error → falhou (+ arquivo_eventos)
```

Idempotent by `hash_sha256` + `source_url` (re-runs skip unchanged files). Every
transition is logged in `arquivo_eventos`.

## AI model routing (per-format specialization, via OpenRouter)

A single seam, `minhaincorp/ai.py`, exposes `classify(file)`, `extract(file)`,
`summarize(...)`. Internally a **`ModelRouter`** selects the model by `(format, task)`
from configuration. **Native parsers run first** to ground and cut cost; AI is used
for vision, messy extraction, summarization and ambiguous classification. Exact model
IDs are pinned at implementation and overridable by config; the table below is the
recommended default routing.

| Format | Native step first | AI model class (via OpenRouter) | Tasks |
|---|---|---|---|
| image / render | mime/size | strong **vision** model | classify, caption, detect typology |
| floor plan (planta) | — | **OCR-strong vision** model | classify, read room/area labels |
| pdf (text) | pdfminer extract text | capable **text** model (e.g. DeepSeek-class) | summary, structured fields |
| pdf (scanned) | detect no text layer | **OCR/vision** model | text + summary |
| ppt/pptx | python-pptx (text+imgs) | text + vision | summary, key facts |
| doc/docx | python-docx (text) | text model | summary, structured fields |
| xls/xlsx/csv (price table) | openpyxl/csv parse | text model only if messy | structured price rows |
| video (mp4) | keyframe sampling + ASR (Whisper-class) | vision on keyframes + text summary | classify, summary |
| classification (high volume) | extension/path heuristics | small fast model when ambiguous | category + confidence |

Provider config: `MINHAINCORP_AI_PROVIDER=openrouter`, `OPENROUTER_API_KEY` (secret,
via `hermes-infra`), plus a routing map (`MINHAINCORP_AI_ROUTES` or a JSON file).
Every extraction persists `modelo_ia`, `prompt_versao` and `confianca` for audit. A
confidence threshold (e.g. `< 0.7`) routes to human review. `# TODO`: wire the real
OpenRouter client; until then `ai.py` ships deterministic stubs so the pipeline and
tests run without network/keys.

## Service & tool surface

**Admin / ingestion** (deterministic service functions + CLI; not LLM tools):

| Need | Function |
|---|---|
| create/update developer, region, project | `developers.upsert` · `regioes.upsert` · `empreendimentos.upsert` |
| import structured JSON | `importacao.importar_json(payload)` |
| ingest files from URL/folder | `ingestao.ingerir(source_ref)` |
| classify / extract | `ingestao.classificar(arquivo_id|lote)` · `ingestao.extrair(...)` |
| human review | `revisao.listar(filtro)` · `revisao.resolver(id, decisao)` |

**Agent tools** (`minhaincorp_*`, typed, grounded):

| Need | Tool |
|---|---|
| list searchable files by developer/project/category | extend `minhaincorp_midia` → file repository, filter by `categoria` |
| retrieve a file for WhatsApp delivery | `minhaincorp_arquivo` → storage ref/URL (`# TODO`: binary send via bridge) |
| search extracted knowledge for broker questions | `minhaincorp_buscar_conhecimento(query, empreendimento?)` → FTS over `extracoes`, returns grounded summary + source file |

Existing tools (`consultar` / `forma_pagamento` / `listar`) remain the structured
truth for price/availability.

## Project layout (existing conventions)

```text
minhaincorp/
  ai.py                       # single AI seam + ModelRouter (stub now, # TODO real)
  services/  regioes.py  arquivos.py  extracoes.py
             importacao.py  ingestao.py  revisao.py
  ingest/    storage.py (backends)  sources.py (adapters)
             classify.py  extract.py  pipeline.py
schemas.py / tools.py         # new agent tools
scripts/   ingest_cli.py  import_json.py     # admin/ops
migrations/ 0003..0006.sql    # idempotent
tests/     mocked db + in-memory storage + AI stub
```

## Phases (what is done now vs later)

**Phase 1 — foundation, no real AI (implement now):**
- [ ] Migrations `0003`–`0006`.
- [ ] `StorageBackend` (localfs) + `SourceAdapter` (http/local; FileHub stub).
- [ ] `importacao.importar_json` (validated, idempotent upsert).
- [ ] Deterministic classifier (heuristics) + `needs_review` plumbing.
- [ ] File registration + `minhaincorp_arquivo` retrieval tool.
- [ ] `ai.py` seam with stubs; `extracoes` writable; FTS column populated.
- [ ] Tests (mocked db, in-memory storage, AI stub).

**Phase 2 — real AI + knowledge search:**
- [ ] OpenRouter `ModelRouter` per-format wiring; native parsers (pdf/docx/pptx/xlsx).
- [ ] AI classify/extract with confidence → review; promotion-after-approval to
      canonical `unidades`/`condicoes_pagamento`.
- [ ] `minhaincorp_buscar_conhecimento` (Postgres FTS).

**Phase 3 — scale & delivery:**
- [ ] pgvector semantic search.
- [ ] WhatsApp binary delivery via bridge (document/contactMessage).
- [ ] Developer self-service ingestion API; productionized `FileHubAdapter`.

## Acceptance criteria

- [ ] EBM is representable as the developer (singleton incorporadora).
- [ ] "Goiânia e Região Metropolitana" is representable as a region.
- [ ] "Rota Panorama" is representable as a project linked to that region.
- [ ] Files from the Rota Panorama folder can be registered (metadata + stored copy).
- [ ] Each file is classified into a usable category or marked `desconhecido_revisar`
      / `review_status=pendente`.
- [ ] Extracted summaries and structured data can be stored (`extracoes`).
- [ ] The backend exposes enough for the assistant to answer and to send the original
      file when requested.
- [ ] The solution is extensible to other developers via new `SourceAdapter`s and the
      versioned JSON import — no schema change per developer.

## Open questions

- FileHub: is there a documented JSON API behind `?path=`, or is an export needed?
  (`FileHubAdapter` detail.)
- Video extraction depth for v1: full ASR + keyframe vision, or summary-from-metadata
  only until volume justifies it?
- Exact OpenRouter model IDs per format (pinned at implementation).

## Related

- `rfcs/RFC-0003-minhaincorporadora-persona-model-correction.md` — the assistant that
  consumes this knowledge.
- `rfcs/RFC-0004-deploy-managed-home-soul.md` — deploy/tenant conventions.
- `decisions/2026-06-25-minhaincorporadora-content-backend.md`.
