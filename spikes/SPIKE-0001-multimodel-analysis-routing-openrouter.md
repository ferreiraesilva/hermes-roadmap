# Spike: Roteamento multi-modelo (imagem/vídeo/documento) via OpenRouter

## Status

Done (investigado nesta sessão; recomendação abaixo)

## Product

MinhaIncorporadora

## Related Epic or Initiative

EPIC-0003 (Track B — Ingestão de Mídia e Documentos). A classificação/enriquecimento
da Track B precisa do "melhor modelo por modalidade".

## Question

Para analisar conteúdo com o melhor modelo por modalidade (um bom para **imagem**,
outro para **vídeo**, outro para **documento/PDF**) via OpenRouter — devemos usar
**vários profiles do Hermes** (um por modelo), ou existe caminho melhor?

## Context

Leonardo assinou o OpenRouter justamente para ter acesso a vários modelos e aumentar
a profundidade da análise. A intuição inicial foi "vários profiles, um por modelo".
Antes de implementar, precisamos entender o que o Hermes realmente oferece (princípio:
checar o manual, não advinhar).

## Research Scope

- O que é um profile no Hermes e o que ele carrega.
- Como o Hermes seleciona modelo/provider (default, por chamada, fallback).
- Como o OpenRouter expõe múltiplos modelos.

## Findings (do manual / CLI do Hermes)

1. **Profile = um bot/identidade/container** com **um** modelo+provider *default*
   (`hermes model` = "Select default model and provider"). Profile carrega persona,
   conversa, token de plataforma — **não** é o mecanismo de "variedade de modelo".
2. **Seleção por chamada existe:** o CLI aceita `-m MODEL` e `--provider` por
   invocação. Logo, **um mesmo profile** pode chamar modelos diferentes por chamada.
3. **`hermes fallback`** = cadeia tentada **quando o primário falha** (rate-limit,
   overload, conexão). É **confiabilidade**, não roteamento por tipo de conteúdo.
4. **OpenRouter = um provider** com `base_url` único; os "vários modelos" são apenas
   **nomes de modelo diferentes** — selecionáveis por chamada.

Conclusão: **roteamento por modalidade não é função de profile.** É uma decisão de
aplicação ("este arquivo é imagem → use o modelo X"), resolvida escolhendo o **nome
do modelo por chamada** no mesmo provider (OpenRouter).

## Recommendation

- **NÃO** criar um profile por modalidade. Profile por modalidade = N containers/bots
  só pra trocar o nome do modelo — peso e complexidade sem retorno.
- **Roteamento por modalidade vive no ingestor** (serviço Track B): uma tabela
  `modalidade → modelo` e chamadas diretas à API do OpenRouter (determinístico, fácil
  de cachear por hash, ideal pra batch). Alternativa equivalente: o ingestor chama
  `hermes -m <modelo> --provider openrouter` por asset — mas a chamada HTTP direta é
  mais controlável pra processamento em lote.
- **Fallback (confiabilidade)** é ortogonal e útil: cada modalidade pode ter um
  modelo alternativo se o primário estiver indisponível.
- **Um único profile do Hermes** continua valendo como **console do operador/monitor**
  (avisar "chegaram 2 empreendimentos", "10 imagens novas no prédio B", "erro X com
  RCA") — agentic onde agrega valor, fora do laço determinístico de classificação.

## Timebox

Concluído (investigação + recomendação).

## Findings → Decisão

Tabela de modelos por modalidade fica como **config do ingestor**, não como profiles.
Documentar a tabela (imagem/vídeo/doc → modelo) quando o EPIC-0003 entrar em execução.

## Follow-up Issues

- [ ] Definir a tabela `modalidade → modelo OpenRouter` (qual modelo p/ imagem, vídeo,
      PDF) no EPIC-0003.
- [ ] Vídeo: avaliar mini-pipeline (keyframes + transcrição de áudio) antes do modelo,
      pois "entender vídeo" puro é caro.
- [ ] Cache por hash: nunca reanalisar asset inalterado (economia no OpenRouter).
