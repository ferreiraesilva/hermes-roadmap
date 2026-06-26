# [EPIC] Campaign Awareness — o bot avisa o corretor das campanhas ativas

## Status

Draft (visão de produto — não agendado). É o **diferencial flagship** do
MinhaIncorporadora.

## Product

MinhaIncorporadora

## Parent Initiative

— (independente; é o primeiro e mais importante dos diferenciais de venda)

## Problem

Hoje o corretor **não sabe** quais campanhas/promoções estão ativas. Quem sabia era
o Gerente de Vendas, que avisava de boca, no grupo de WhatsApp, ou numa "peça
digital" perdida numa pasta. Resultado: o corretor perde **argumentos de fechamento**
poderosos e perecíveis.

Exemplo real (palavras do Leonardo): o corretor pergunta sobre o **Rota Panorama** e
existe uma campanha *"aceitamos seu carro a 100% da tabela FIPE como entrada"*. Isso
é uma novidade e um ótimo gatilho de venda — mas se ninguém avisa, a campanha morre
na pasta. Outro exemplo, já presente no FileHub da EBM: a peça **"Assinou ganhou"**
do Now 111, e os **"Convites abertura de vendas"** do Rota Panorama (SAVE THE DATE,
"FALTAM 2 DIAS", "É AMANHÃ") — tudo isso é ação de venda com **prazo**.

Campanha é, por natureza, **temporal**: tem início, fim, e some quando expira. Um bot
que serve preço fixo mas ignora campanha está deixando na mesa o argumento mais
fresco que existe.

## Objective

O assistente conhece as campanhas **ativas e válidas** e:
1. **avisa proativamente** quando uma campanha se aplica ao que o corretor está
   perguntando (ele pergunta do Rota Panorama → o bot já emenda a campanha ativa);
2. **lista sob demanda** ("tem alguma campanha agora?", "o que tem de promoção em
   Goiânia?");
3. **nunca** menciona campanha expirada (correção > recall — vergonha de vender algo
   que acabou).

Esse comportamento proativo é o que diferencia o produto de um simples FAQ.

## Scope

- **Entidade Campanha** (estruturada, não um arquivo solto):
  `{ id, titulo, oferta/descricao, alvo (empreendimentos[] ou "toda a incorporadora"),
  inicio, fim, prioridade, material[] (imagens/PDF/vídeo da campanha), ativo }`.
- **Injeção proativa**: ao responder sobre um empreendimento, o bot verifica se há
  campanha ativa para ele e **emenda** ("🔥 Campanha ativa até DD/MM: …").
- **Listagem sob demanda**: por empreendimento, por praça, ou geral.
- **Validade**: filtro determinístico por data (a campanha some sozinha ao expirar).
- **Material da campanha**: entregue como mídia nativa (ver Track B / RFC-0005).

## Out of Scope

- O **portal/API** onde a incorporadora cadastra/atualiza campanhas — isso é a API
  padrão comercial (campanha é fato mantido pela incorporadora, como preço). Aqui é
  só o **consumo + comportamento** do bot.
- **QR / convites de evento** — são do EPIC-0002 (eventos). Campanha ≠ evento, embora
  uma campanha possa anunciar um evento.

## User Flow

1. Corretor: *"me fala do Rota Panorama"*.
   Bot: responde (empreendimento/unidades via Track A) **e** emenda:
   *"🔥 Aproveite: até 30/06 o Rota Panorama aceita seu carro a 100% da tabela FIPE
   como entrada. Quer a peça pra mandar pro cliente?"* → oferece o material da campanha.
2. Corretor: *"tem alguma promoção em Goiânia agora?"*.
   Bot: lista as campanhas ativas das praças/empreendimentos de Goiânia, com validade.
3. Campanha expira → some automaticamente; o bot para de mencionar.

## Target Repository

`hermes-minhaincorporadora` (consumo/comportamento) + a API/portal padrão (cadastro).

## Dependencies

- RFC-0005 (backend de conteúdo) — o material da campanha é mídia entregável.
- API/portal comercial padrão — fonte de verdade das campanhas (título, oferta,
  validade, alvo). Campanha é dado mantido pela incorporadora, não inferido de PDF.

## User Stories

- [ ] Como corretor, ao perguntar de um empreendimento, sou avisado das campanhas
      ativas dele sem precisar pedir.
- [ ] Como corretor, posso listar as campanhas ativas (geral, por praça, por
      empreendimento).
- [ ] Como corretor, recebo o **material** da campanha (imagem/vídeo/PDF) pra repassar
      ao cliente.
- [ ] Como sistema, nunca mostro campanha fora do período de validade.

## Technical Tasks

- [ ] Modelar tabela `campanhas` (+ vínculo com empreendimentos) no schema minhaincorp.
- [ ] Ferramenta `minhaincorp_campanhas` (listar/filtrar ativas por alvo e data).
- [ ] Injeção proativa no fluxo de resposta sobre empreendimento (hook/contexto).
- [ ] Entrega do material da campanha via a camada de mídia (Track B).
- [ ] Endpoint/porta no portal padrão pra incorporadora cadastrar campanha.

## Acceptance Criteria

- [ ] Perguntar de um empreendimento com campanha ativa → o bot menciona a campanha e
      oferece o material, com a data de validade.
- [ ] Campanha expirada nunca aparece.
- [ ] Listagem sob demanda funciona por empreendimento, praça e geral.
