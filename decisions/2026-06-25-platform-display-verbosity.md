# Display e verbosidade por plataforma

## Status

Aceito em 2026-06-25.

## Contexto

O gateway do Hermes pode enviar mensagens intermediárias durante uma execução:
chamadas de ferramentas, comandos de terminal, consultas de memória/sessão e
status de progresso. Em canais como WhatsApp, essas mensagens viram conteúdo
permanente na conversa e competem com a resposta final do agente.

No HML pessoal do Leonardo, isso apareceu como mensagens do tipo
`session_search`, `terminal` e chamadas de plugin sendo enviadas no WhatsApp. A
transcrição de áudio deve continuar visível, mas o console/progresso interno não.

Também existe o comando `/sethome`, que muda o diretório/home operacional de uma
sessão. Configurações de display do canal não devem ser colocadas nesse local,
porque são política do container/profile e precisam sobreviver a redeploy.

## Decisão

Verbosidade de mensagens intermediárias deve ser configurada por plataforma no
profile do `hermes-infra`, usando a superfície nativa do Hermes:

```yaml
display:
  platforms:
    whatsapp:
      tool_progress: off
```

Para o HML pessoal do Leonardo, o WhatsApp fica com `tool_progress: off`. Isso
suprime chamadas de ferramenta no chat, preservando a transcrição de áudio e as
mensagens finais do agente.

## Consequências

- O comportamento é reproduzível por deploy e não depende de ajuste manual no
  container.
- O `/sethome` continua restrito ao contexto operacional da sessão; não vira
  ponto de configuração de canal.
- Outros canais podem escolher `off`, `new`, `all` ou `verbose` conforme o uso.
- Reclamações futuras sobre ruído em canais externos devem ser tratadas primeiro
  como configuração de `display.platforms`, antes de alterar plugins.
