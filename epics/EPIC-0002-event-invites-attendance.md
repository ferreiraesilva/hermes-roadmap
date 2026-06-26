# [EPIC] Eventos & Convites — convite por corretor com QR único e presença premiada

## Status

Draft (visão de produto — não agendado).

## Product

MinhaIncorporadora

## Parent Initiative

Relacionado à INITIATIVE-0001 (Club do Corretor) — presença em evento é uma das
principais fontes de pontos de fidelidade.

## Problem

Incorporadora vende em **lançamento**, e lançamento é **evento**: abertura de vendas,
meeting, coquetel, premiação. A EBM faz isso de verdade — no FileHub do **Rota
Panorama** existem as pastas *"Convites abertura de vendas"* (com "SAVE THE DATE",
"FALTAM 2 DIAS", "É AMANHÃ", convite animado), *"Convite repique"*, *"Fotos abertura
de vendas"* (181 fotos), *"Fotos meeting"* (mais de 580 fotos) e **"Premiação
Lançamento"** (PDF com os prêmios).

Hoje convidar o corretor é manual e sem rastreio: manda-se uma imagem genérica num
grupo, não se sabe **quem vai**, e o credenciamento na porta é no improviso. Como o
evento dá **brindes top** (celular, churrasqueira, MacBook, tablet), saber quem é
quem na entrada importa — e o convite genérico não identifica ninguém.

## Objective

Transformar o convite num **ativo identificado por corretor**: uma **imagem linda com
um QR code único** por corretor, que:
1. é entregue pelo assistente ao corretor certo (ele já está identificado pelo canal);
2. é **lido na entrada do evento** pra credenciar/identificar quem chegou;
3. alimenta presença (e, no futuro, pontos de fidelidade — INITIATIVE-0001).

## Scope

- **Entidade Evento**: `{ id, titulo, empreendimento/lançamento, data, local, descricao,
  material[] (convites, save the date, premiação), ativo }`.
- **Convite por corretor**: geração de uma **peça visual com QR único** (token
  irreproduzível) por (evento × corretor).
- **Entrega**: o assistente manda o convite ao corretor (imagem nativa).
- **Credenciamento**: app/fluxo que **lê o QR na porta**, valida e registra presença
  (uso único — não pode ser repassado/reaproveitado).
- **Material do evento**: save-the-date, repique, premiação — entregues como mídia.

## Out of Scope

- Programa de fidelidade em si (pontos/troca) — é a INITIATIVE-0001; aqui só
  registramos **presença** como insumo.
- Gestão de campanha promocional — é o EPIC-0001.

## User Flow

1. Incorporadora cria o evento (ex.: "Abertura de vendas Rota Panorama — 30/06").
2. Assistente entrega a cada corretor convidado uma **imagem de convite com QR único**.
3. Corretor chega no evento; o QR é lido na porta → presença registrada (uso único).
4. (Futuro) presença vira pontos no Club do Corretor.
5. Após o evento, o assistente pode entregar fotos/material ("manda as fotos da
   abertura de vendas do Rota Panorama").

## Target Repository

`hermes-minhaincorporadora` (entrega/consulta) + um serviço de geração/validação de
QR + app de leitura na porta (a definir).

## Dependencies

- RFC-0005 (conteúdo) — convites, save-the-date, premiação e fotos do evento são mídia.
- EPIC-0001 (campanhas) — um evento costuma ser anunciado por uma campanha.
- Serviço de QR (geração de token único + leitura/validação) — provavelmente novo.

## User Stories

- [ ] Como corretor, recebo do assistente um convite com **meu** QR pro lançamento.
- [ ] Como recepção do evento, **leio o QR** e identifico o corretor na entrada.
- [ ] Como sistema, garanto que cada QR é **único e de uso único** (sem repasse).
- [ ] Como corretor, peço ao assistente o material/fotos do evento depois.

## Technical Tasks

- [ ] Modelar `eventos` e `convites` (token QR por evento×corretor) no schema.
- [ ] Geração da peça visual de convite com QR embutido.
- [ ] Serviço de validação/credenciamento (leitura na porta, marca presença, uso único).
- [ ] Ferramenta `minhaincorp_evento` (consultar evento, pedir convite, pedir material).
- [ ] Registro de presença como evento auditável (insumo p/ fidelidade).

## Acceptance Criteria

- [ ] Corretor recebe um convite com QR exclusivo dele.
- [ ] QR lido na porta credencia e marca presença; tentativa de reuso é bloqueada.
- [ ] Material e fotos do evento podem ser solicitados depois pelo assistente.
