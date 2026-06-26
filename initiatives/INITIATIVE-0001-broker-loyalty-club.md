# Initiative: Club do Corretor — programa de fidelidade ("Club EBM", "Club City", …)

## Status

Draft (visão de longo prazo — não agendado). É a maior das ideias; provavelmente uma
iniciativa que se desdobra em vários epics.

## Product

MinhaIncorporadora

## Summary

Um **programa de fidelidade para corretores**, white-label por incorporadora —
"Club EBM", "Club City", e assim por diante. O corretor **acumula pontos** por ações
que interessam à incorporadora e **troca esses pontos** por benefícios que ajudam o
corretor a vender mais. O objetivo é criar um **vínculo** entre o corretor e a
incorporadora: o corretor que está no Club vende mais daquela incorporadora porque é
reconhecido e apoiado por ela.

A grande sacada é que o assistente (MinhaIncorporadora) já está na palma da mão do
corretor, no WhatsApp — então ele é o **canal natural** do Club: mostra saldo de
pontos, avisa de oportunidades de ganhar pontos, e processa as trocas.

## Problem

O corretor é autônomo e vende de **várias** incorporadoras. Falta um mecanismo que o
incentive a priorizar uma incorporadora e a se engajar (ir a eventos, fazer
treinamento, vender mais). Hoje esse vínculo é informal e depende do relacionamento
pessoal do Gerente de Vendas — não escala.

## Objective

Criar e operar um programa de pontos+recompensas que aumente o **engajamento e a
fidelidade** do corretor com a incorporadora, operado através do assistente.

## Target Users

Corretores das imobiliárias parceiras (ganham/trocam pontos) e a incorporadora
(define ações pontuáveis e o catálogo de recompensas).

## Scope

- **Acúmulo de pontos** por ações, por exemplo:
  - ir a um **evento** (presença via QR — EPIC-0002);
  - participar de **treinamento**;
  - **vender** (a maior fonte de pontos);
  - outras ações que a incorporadora definir.
- **Catálogo de recompensas** trocáveis por pontos, por exemplo:
  - **dia top de plantão** (o melhor plantão de vendas);
  - **empréstimo de material de apoio e distribuição** (água, e até uma pessoa pra
    distribuir panfleto pro corretor);
  - **produção de material de marketing** nas redes sociais do corretor;
  - **entrada preferencial** em eventos e **sorteios exclusivos**.
- **Operação pelo assistente**: consultar saldo, ver como ganhar/gastar, solicitar
  troca; tudo no WhatsApp.
- **White-label**: o nome e o catálogo são por incorporadora ("Club EBM", "Club City").

## Out of Scope

- Implementação imediata. É visão; entra depois que o núcleo (conteúdo + comercial +
  campanhas + eventos) estiver de pé.
- Integração financeira/contábil de prêmios de alto valor (a definir).

## Success Criteria

- Corretores ativos no Club vendem mais daquela incorporadora (engajamento medível).
- Pontos e trocas são **auditáveis e à prova de fraude**.
- A incorporadora consegue configurar ações pontuáveis e o catálogo sem depender de nós.

## Related Epics

- [ ] EPIC: Ledger de pontos do corretor (acúmulo, saldo, extrato).
- [ ] EPIC: Catálogo de recompensas + fluxo de troca/resgate.
- [ ] EPIC: Integração com presença em evento (EPIC-0002) e com vendas como fontes de
      pontos.
- [ ] EPIC: Configuração white-label do Club por incorporadora.

## Dependencies

- EPIC-0002 (eventos) — presença é fonte de pontos.
- Sinal de **venda** confirmada (de onde vem? CRM da incorporadora? a API padrão?).
- Backend de conteúdo (RFC-0005) e a API/portal padrão.

## Risks

- **Integridade dos pontos** (fraude, duplicidade) — precisa de ledger confiável.
- **Custo das recompensas** (prêmios físicos, pessoas pra distribuir panfleto) — modelo
  de custo por incorporadora.
- **Escopo gigante** — risco de virar um produto inteiro; fatiar bem em epics.

## Open Questions

- De onde vem o sinal de **venda** (a maior fonte de pontos)?
- O Club é um serviço compartilhado entre incorporadoras ou isolado por tenant?
- Quem banca e logística das recompensas físicas?

## Target Repositories

- `hermes-minhaincorporadora` (canal/assistente)
- Um serviço de Club/fidelidade (a definir — possivelmente novo)
