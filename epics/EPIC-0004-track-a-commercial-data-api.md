# [EPIC] Track A — Dado comercial: planilha de onboarding + API/portal padrão

## Status

Draft (piloto: importação de planilha; futuro: API + portal)

## Product

MinhaIncorporadora

## Parent Initiative

— (núcleo: o fato duro que o bot nunca inventa)

## Problem

Preço, disponibilidade, metragem, forma de pagamento, praças e empreendimentos são
**fato comercial** — fonte única e autoritativa, nunca inferida de documento. A
incorporadora não tem (nem precisa ter) API; **nós** somos donos do padrão, e ela
**consome/alimenta** (clientes grandes integram; pequenos usam um **portal** que
preenchemos). Para o **piloto da EBM**, ainda não há API: o dado entra por **planilha**.

## Objective

Ter o fato comercial num store estruturado que nós controlamos, alimentado de forma
padronizada — começando por importação de planilha (piloto) e evoluindo para a **API
REST padrão + portal**, igual para todos os containers.

## Scope

- **Planilha de onboarding** (já gerada: `onboarding/dados_ebm.xlsx`) com abas Praças,
  Empreendimentos, Unidades, FormaPagamentoPadrao. Leonardo preenche o comercial.
- **Migration de campos novos** que a planilha pede e o schema ainda não tem:
  - empreendimentos: `tipo` (residencial/comercial/misto), `latitude`, `longitude`
  - unidades: `torre_bloco`, `escaninho`, `tags`, `itens_inclusos`, `preco_promocional`
- **Importador planilha → JSON canônico → DB** (idempotente, estado completo).
- **Id estável de empreendimento** (slug/código) — é a **chave de junção** da mídia
  (Track B) ancorar, sem depender de nome com acento/NFD.
- **API REST padrão** (futuro): CRUD de praça/empreendimento/unidade/preço/
  disponibilidade/forma de pagamento — a incorporadora alimenta.
- **Portal** (futuro): UI fina sobre a API pra cliente sem TI preencher.

## Out of Scope

- Construir API/portal **agora** (YAGNI no piloto — o bot lê o DB, não importa se o
  dado entrou por planilha ou API). API/portal vêm após 1–2 clientes validarem o
  contrato.
- Mídia/documento (Track B — EPIC-0003) e campanhas/eventos (EPIC-0001/0002).

## User Flow (piloto)

Leonardo preenche `dados_ebm.xlsx` → importador valida e carrega no `db_incorporadora_ebm`
→ o bot passa a responder preço/disponibilidade reais da EBM (hoje responde "não há
empreendimentos" porque o tenant `ebm` está vazio).

## Target Repository

`hermes-minhaincorporadora` (schema + importador) e, no futuro, o serviço da API/portal.

## Dependencies

- RFC-0005 (backend de conteúdo) e o schema `minhaincorp`.
- A planilha de onboarding (gerador `onboarding/_gen_planilha_ebm.py`).

## User Stories

- [ ] Como operador, importo a planilha preenchida e o banco do tenant é populado.
- [ ] Como corretor, recebo preço/disponibilidade/forma de pagamento **reais** (fato).
- [ ] (Futuro) Como incorporadora, mantenho meus dados pela API/portal padrão.

## Technical Tasks

- [ ] Migration: campos novos em empreendimentos (tipo, lat, long) e unidades
      (torre_bloco, escaninho, tags, itens_inclusos, preco_promocional).
- [ ] Importador planilha (.xlsx) → JSON canônico → upsert (mapear status/disp. PT,
      slugs como id, forma_pagamento_padrao → coluna do empreendimento).
- [ ] Validações (ids existem, enums válidos, números limpos).
- [ ] (Futuro) Desenho da API REST padrão + autenticação por incorporadora.
- [ ] (Futuro) Portal de preenchimento.

## Acceptance Criteria

- [ ] Planilha preenchida importa sem erro e popula praças/empreendimentos/unidades/
      forma de pagamento do tenant EBM.
- [ ] Bot responde fato comercial real da EBM, ancorado no id de empreendimento.
- [ ] Campos novos disponíveis no schema (migration aplicada).
