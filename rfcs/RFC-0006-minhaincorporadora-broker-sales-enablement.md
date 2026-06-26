# RFC-0006: MinhaIncorporadora — Broker Sales-Enablement Differentiators

## Status

Proposed — future (not scheduled). Campaign awareness is the highest-priority of
the three and a likely flagship differentiator; events and loyalty are later.

## Product

MinhaIncorporadora

## Summary

The assistant is not only a Q&A over empreendimentos — it is the broker's **selling
arsenal**. Beyond unit data (price/availability) and per-empreendimento media
(RFC-0005), it must also: proactively surface **active campaigns/promotions**,
distribute **institutional/brand material** (to sell the developer's solidity),
deliver **event invites with a unique QR code**, and eventually run a **loyalty
program** ("Club <Developer>").

This RFC corrects an earlier framing where institutional/campaign/event folders in
a developer's repository were treated as "marketing noise" to exclude. They are
**first-class selling content**, just not commercial unit data — so they belong in
the content layer (RFC-0005), some with their own structured model.

## Motivation

A broker sells an **off-plan** property: the client pays ~60% of the value over
~4 years, betting the developer won't go bankrupt and take their money. So the
broker needs to sell the **company's credibility**, not just the apartment —
institutional material is core, not optional.

Campaigns are time-bound sales arguments the broker often doesn't know about. If a
broker asks about "Rota Panorâmica" and there is an active campaign ("we accept a
car at 100% of the FIPE table"), the assistant should **proactively** say so — it is
a fresh, strong closing argument. Surfacing campaigns proactively is expected to be
one of the biggest differentiators of MinhaIncorporadora.

## Scope

Three capabilities, in priority order:

### 1. Campaign / Promotion Awareness (flagship)
A **campaign** is a structured, time-bound entity, not just a file:
`{ titulo, oferta/descricao, empreendimentos[] (or developer-wide), inicio, fim,
material[] (images/PDF/video) }`. The assistant **proactively** tells the broker
when a campaign applies to what they asked about, and can list active campaigns on
demand. Deserves its own capability/"chapter" in the tool. Must never surface an
expired campaign (time-bound; source of truth in the structured store).

### 2. Institutional / Brand material
Developer-level content (institutional deck, brand assets, "the company is solid"
proof, institutional video) anchored at the **incorporadora** level (not a single
empreendimento). The broker uses it to sell the company. First-class content class
in the RFC-0005 taxonomy.

### 3. Events + QR invites
On launches, developers hold events and invite brokers. The assistant delivers a
**per-broker invite** — a polished image carrying a **unique QR code** — scanned at
the event entrance. Events give top prizes (phones, grills, MacBook, tablet), so
attendance/identification matters. Needs: event entity, per-broker invite/QR
generation, and a scan/redeem flow.

### 4. Loyalty program — "Club <Developer>" (e.g., Club EBM / Club City)
Brokers accumulate points (attending events, training, and especially **sales**) and
redeem them for perks offered by the developer: a premium "plantão" day, loan of
support/distribution material (water, a person to hand out flyers), production of
social-media marketing for the broker, preferential event entry, and exclusive
raffles.

## Out of Scope

No implementation, schema, or migration now. This is vision/backlog. Sequencing:
finish the RFC-0005 content backend and the EBM pilot first; promote **Campaign
Awareness** to its own implementation RFC/epic when scheduled.

## Functional Requirements

- Campaign: model active campaigns with validity window and target empreendimentos;
  proactively mention applicable active campaigns in answers; list on demand; never
  show expired ones.
- Institutional: store/classify developer-level material; broker can request "material
  institucional / sobre a EBM".
- Events: model events; generate a unique per-broker QR invite (image); scan/redeem.
- Loyalty: points ledger per broker; catalog of redeemable perks; redemption flow.

## Non-Functional Requirements

- Reliability: expired campaigns must never be surfaced (correctness > recall).
- Security: QR invites must be unique and non-forgeable; loyalty points tamper-proof.
- Observability: campaign/event activity and point movements auditable.
- Maintainability: reuse RFC-0005 content pipeline; campaigns/events as structured
  data fed via the same standard API / portal (not parsed from documents).

## Dependencies

- RFC-0005 (content ingestion & knowledge backend) — campaigns/events/institutional
  are content + structured data on top of it.
- The standard commercial API / portal (developers feed campaigns there, same as
  prices) — campaigns are a developer-maintained, time-bound fact.

## Risks

- **Stale campaigns** surfaced after expiry → must be strictly time-bound.
- QR invite forgery / double-use at events.
- Loyalty point integrity and abuse.
- Scope creep: this is a large vision; keep campaign-awareness as the first, focused
  slice.

## Open Questions

- Source of truth for campaigns: standard API, portal, or a campaigns sheet/folder?
- Who owns QR generation and the event scan app?
- Loyalty backend: in-product, or a separate "Club" service shared across developers?

## Acceptance Criteria

- Vision recorded and discoverable in the roadmap.
- When scheduled, **Campaign Awareness** is promoted to its own implementation
  RFC/epic with a concrete data model and proactive-surfacing behavior.
