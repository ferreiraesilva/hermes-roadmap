# [EPIC] Track B — Ingestão e classificação de mídia e documentos (piloto EBM)

## Status

Draft (pronto pra detalhar; piloto na EBM)

## Product

MinhaIncorporadora

## Parent Initiative

— (núcleo do produto; habilita campanhas/eventos — EPIC-0001/0002 — e o atendimento
com material real)

## Problem

A incorporadora guarda o material num repositório próprio e bagunçado (a EBM usa o
"FileHub" em `arquivos.ebm.com.br`, navegado por `?path=`/`file_browser.php`). O bot
precisa **achar e entregar o arquivo certo** ("manda a foto da cozinha", "que vídeos
tem do Rota Panorama") sem reler tudo a cada pergunta e **sem inventar**. O conteúdo
não tem etiqueta: o significado está no **nome da pasta** e no **nome do arquivo**, e
no **conteúdo** (imagem/vídeo/PDF) — cabe à IA capturar isso.

## Objective

Pipeline que **adquire, classifica, estrutura e expõe** o conteúdo de cada
incorporadora como conhecimento buscável + arquivo entregável nativo, alinhado ao
contrato canônico (RFC-0005), reaproveitável por cliente via **adapter de fonte**.

## Scope

Três estágios (ver discussão de arquitetura): **1) Adquirir** (específico da fonte) →
**2) Enriquecer → Contrato** (IA, agnóstico) → **3) Carregar** (determinístico). O
contrato fica **materializado** entre 2 e 3 (revisável; classifica-se uma vez).

### Taxonomia de mídia (aterrada em Now 111 + Rota Panorama)
Pasta = categoria; nome do arquivo = sub-tag; conteúdo confirma/enriquece.

| Classe | Pastas reais | Sub-tag no arquivo |
|---|---|---|
| Produto | Fotos decorado, Perspectivas, Imagem aérea, Plantas, Book, Manual do corretor, Vídeos (teaser/produto/tour/VR) | `Cozinha`, `Perspectiva fachada/gourmet`, `Planta 2Q/studio` |
| Institucional/marca | Marca (logos), Apresentação Meeting | — |
| Campanha | Peças digitais (feed/story), "Assinou ganhou" | — |
| Evento | Convites abertura de vendas, Convite repique, Fotos abertura/meeting, Premiação | — |
| Comercial (ignorar p/ fato) | Tabela/TABELA (PDF datado) | preço vem da Track A |

## Out of Scope

- Extrair preço/disponibilidade de documento (vem da Track A / API — EPIC-0004).
- O modelo estruturado de campanha/evento em si (EPIC-0001/0002) — aqui entregamos o
  **material** dessas classes; a lógica temporal/QR é dos outros epics.

## User Flow

Corretor: "manda a foto da cozinha do Now 111" → bot acha o asset classificado
(categoria=foto-decorado, tag=cozinha) e **envia o arquivo nativo**. "Que vídeos tem
do Rota Panorama?" → "teaser, produto, localização, tour VR" → corretor escolhe →
bot entrega.

## Target Repository

`hermes-minhaincorporadora` (+ um serviço/worker de ingestão; ver SPIKE-0001).

## Dependencies

- RFC-0005 (design do backend de conteúdo).
- SPIKE-0001 (roteamento multi-modelo via OpenRouter — sem profiles por modalidade).
- EPIC-0004 (Track A) — o **id estável de empreendimento** é a chave em que a mídia
  ancora (resolve o problema de casar nome com acento/NFD).

## User Stories

- [ ] Como corretor, peço uma foto/planta/vídeo/PDF por assunto e recebo o **arquivo**
      (não link), nativo no WhatsApp.
- [ ] Como corretor, pergunto "que material tem do empreendimento X?" e recebo a lista
      por categoria.
- [ ] Como operador, sou avisado do que foi ingerido/classificado e dos erros (RCA).
- [ ] Como sistema, nunca entrego material de empreendimento errado (ancora no id).

## Technical Tasks

- [ ] **Adapter de fonte EBM/FileHub**: listar via `file_browser.php?path=`, baixar
      bytes (sem links). Genérico por cliente no futuro.
- [ ] **Aquisição + dedup por hash** + storage local (`files_dir` por instância; object
      store no futuro). Nunca rebaixar/reanalisar asset inalterado.
- [ ] **Sniff de tipo por conteúdo (magic bytes)** — há arquivos **sem extensão** (17
      perspectivas, imagem aérea e vídeos do Rota Panorama). Não confiar na extensão.
- [ ] **Pular parciais/temporários** (`.pptx.part` apareceu no Rota Panorama).
- [ ] **Classificação/enriquecimento multi-modelo** (imagem/vídeo/PDF → modelo via
      OpenRouter, SPIKE-0001): categoria + tags + **descrição rica** (indexada na FTS) +
      tipo de documento (planta/manual/memorial — todos entregáveis, nenhum vira fato).
- [ ] **Galerias de evento**: pastas com 180–320 fotos (Fotos abertura/meeting) →
      classificar no **nível da pasta + amostragem**, não asset a asset (custo).
- [ ] **Mapeamento pasta→categoria / nome→sub-tag**, com conflito (nome × conteúdo) →
      gate de revisão.
- [ ] **Contrato JSON de mídia** (asset → empreendimento_id, classe, categoria, tags,
      descrição, arquivo) — materializado e revisável.
- [ ] **Loader** (`import_json` → `midias` + campos): idempotente, estado completo.
- [ ] **Gate de revisão humana** (`lotes_importacao`) antes de publicar no banco vivo.
- [ ] **Console/monitor** (1 profile Hermes): relatório de ingestão + RCA de erro.

## Acceptance Criteria

- [ ] Pedir material por assunto retorna o **arquivo nativo** correto, ancorado no
      empreendimento certo.
- [ ] Arquivos sem extensão e parciais são tratados corretamente (sniff/pular).
- [ ] Galerias grandes não estouram custo (classificação por pasta/amostra).
- [ ] Nada de fato comercial inferido de documento.
