# Decision: MinhaIncorporadora searches broadly before narrowing inventory

## Status

Accepted — 2026-06-26

## Context

In the EBM homologation deployment, a broker asked for inventory matching a
typology such as "3 suites" over WhatsApp. The assistant replied that it needed a
specific development before checking availability.

That behavior is wrong for a sales-assistant workflow. Brokers often start with a
buyer constraint — typology, budget, availability status, area or region — and only
later narrow the conversation to one development.

## Decision

MinhaIncorporadora must support broad inventory searches across all developments
when the broker asks for a filter without naming a development.

Examples:

- "Tem 3 suítes?"
- "O que tem reservado?"
- "Tem algo até 900 mil?"

In these cases, the assistant must call the inventory tool without a development,
return matching options grouped by development, and invite the broker to choose one
for deeper details.

It must still ask for a specific development when the request inherently depends on
one, such as payment terms, media/plans, or detailed follow-up on a single option.

## Consequences

- `minhaincorp_consultar` no longer requires `empreendimento` when a filter is
  present.
- The service layer supports a multi-development inventory query grouped by
  development and typology.
- The persona instruction changes from "always ask before consulting" to "do not
  assume; search broadly for broad filter questions, then narrow".
- Typology matching must tolerate common WhatsApp spelling differences, including
  "suite" vs. "suíte", by using structured columns such as `suites` when possible.

## Related Repositories

- `hermes-minhaincorporadora` — tool schema, handler, service, template, persona and
  tests.
