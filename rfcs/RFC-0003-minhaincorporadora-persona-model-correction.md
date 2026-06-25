# RFC-0003: MinhaIncorporadora Persona and Domain Model Correction

## Status

Accepted — implemented on `feature/minhaincorp-v1` (`hermes-minhaincorporadora`)

## Product

MinhaIncorporadora

## Summary

MinhaIncorporadora was built and shipped with a persona that was framed
incorrectly. The assistant was described as a *broker persona* ("persona de
corretor") and, after a first correction, as a standalone *commercial attendant*
("Atendente Comercial"). Both framings miss what the assistant actually is.

The assistant is **the Sales Manager's Assistant** ("Assistente do Gerente de
Vendas"). The broker (corretor) is **who it serves**, not who it impersonates.
The assistant acts on behalf of a specific Sales Manager toward that manager's
brokers, and escalates back to that same Sales Manager when it cannot answer from
the data.

This RFC records the corrected domain model and the changes required to align the
persona, the escalation copy, the documentation and the schema semantics with it.

## Motivation

A broker relays prices and conditions to a real buyer. The credibility of every
answer depends on the assistant occupying the right role. "Broker persona" implies
the bot *is* a broker (it is not — it serves brokers). "Commercial attendant" is a
generic role disconnected from the human responsible for the relationship.

The real-world relationship is: the **Sales Manager** owns the commercial
relationship with a set of partner real estate agencies and their brokers. The
assistant exists to do, at scale and instantly, what that Sales Manager used to do
by hand — answer brokers about developments, send material, and hand off to the
manager when the question leaves the data. Naming the assistant correctly makes the
escalation natural ("talk to *your* Sales Manager") and keeps the bot anchored to a
real, accountable human.

## Corrected Domain Model

```text
Incorporadora (tenant — one per installation)
└── Gerentes de Vendas (Sales Managers — each one handles X agencies)
    └── Assistente do Gerente de Vendas  ← the bot (this product)
        └── Corretores (brokers — each linked to one agency)
```

- A **Sales Manager** (Gerente de Vendas) handles one or more **agencies**
  (imobiliárias).
- An **agency** exists only to link a **broker** to exactly one **Sales Manager**.
  It carries no other behavior in this product.
- A **broker** belongs to one agency. The broker's phone identifies him, activates
  the persona (in `shared` mode), and — through his agency — resolves the Sales
  Manager used for escalation.
- The **assistant** is the Sales Manager's assistant. When the broker is known, the
  assistant knows exactly which Sales Manager it represents (the broker's agency's
  manager) and may name that manager.

This model maps onto the existing schema with no structural redesign: the physical
table `gerentes` already represents the Sales Manager, and `imobiliarias.gerente_id`
already links an agency to one manager. The correction is primarily semantic (what
each entity *is*) plus a constraint that the model now makes mandatory.

## Scope

- Reframe the persona from "Atendente Comercial" / "persona de corretor" to
  **"Assistente do Gerente de Vendas"** in `SOUL.md` and `minhaincorp/persona.py`.
- When the broker (and therefore his Sales Manager) is known, let the assistant
  name the Sales Manager it works for. Keep a generic fallback when unknown.
- Align terminology across the product: "gerente de atendimento" / "gerente
  comercial" → **"Gerente de Vendas"** in services, tool descriptions, templates,
  README, AGENTS and PROGRESS.
- Reframe escalation copy: hand off to the broker's **Sales Manager**.
- Encode the model in the schema via migration `0002`:
  - Document the corrected meaning of `gerentes`, `imobiliarias` and `corretores`
    with column/table comments.
  - Make `imobiliarias.gerente_id` **NOT NULL** (an agency without a Sales Manager
    is meaningless under the corrected model), applied safely and idempotently.
- Update the affected tests to assert the new persona identity.
- Update the catalog description in `/config/product-catalog.md` so the product is
  no longer described as a "broker assistant".

## Out of Scope

- Renaming the physical table `gerentes` to `gerentes_vendas`. The AGENTS.md
  principle "additive, not destructive" applies; the physical name is retained and
  documented as "Gerente de Vendas". A rename can be a separate future task if
  desired.
- Any change to the multi-tenant model, activation modes (`shared` / `dedicated` /
  `telegram_test`), or the zero-invention rule. Those remain exactly as designed.
- New broker-facing capabilities (lead capture, visit scheduling, proactive news).
  Those remain post-MVP candidates and are unaffected by this correction.

## User Flow

1. A broker messages the number. The channel identifies him by phone.
2. The hook resolves the broker → his agency → his Sales Manager, and injects the
   **Assistente do Gerente de Vendas** persona. When the manager is known, the
   persona states which Sales Manager it works for.
3. The assistant answers questions about developments strictly from the
   `minhaincorp_*` tools (price, availability, typology, payment terms, media).
4. When the question leaves the data (negotiation, exception, legal, special
   condition), the assistant escalates: it returns the contact of **the broker's
   Sales Manager** and logs the interaction.

## Functional Requirements

- The persona identifies as the Sales Manager's Assistant, never as a broker and
  never as a generic assistant.
- When the broker and his Sales Manager are known, the persona may name that Sales
  Manager; when unknown, it uses a generic Sales-Management framing.
- Escalation resolves and returns the broker's Sales Manager (name, phone, optional
  email, clickable WhatsApp link).
- All hard facts continue to come exclusively from the `minhaincorp_*` tools.

## Non-Functional Requirements

- **Security**: no new data exposure; escalation still only reveals the manager who
  owns the broker's relationship.
- **Maintainability**: terminology is consistent across code, docs and persona, so
  future contributors are not re-misled by "broker persona" / "attendant".
- **Reliability**: schema constraint (`gerente_id NOT NULL`) is applied only when no
  violating rows exist, and is idempotent.
- **Observability**: the `atendimentos` audit log is unchanged.

## Dependencies

- `hermes-infra` — owns the concrete EBM/City deployments and bot names; no infra
  change is required by this RFC, but the corrected vocabulary should be used in any
  future client-facing description.

## Risks

- Existing rows with `imobiliarias.gerente_id IS NULL` would block the NOT NULL
  constraint. Mitigation: the migration checks first and skips the constraint with a
  clear notice instead of failing the deploy.
- Tests assert persona strings; they are updated in the same change to avoid a red
  build.

## Open Questions

- Should the physical table `gerentes` eventually be renamed to `gerentes_vendas`
  for clarity? Deferred; not required for the correction.

## Acceptance Criteria

- [x] Given a registered broker, when the persona is injected, then it identifies as
      the "Assistente do Gerente de Vendas" and not as a broker or attendant.
- [x] Given a registered broker whose Sales Manager is known, when the persona is
      injected, then it may name that Sales Manager.
- [x] Given a broker asks something outside the data, when the assistant escalates,
      then it returns the broker's **Sales Manager** contact and logs the interaction.
- [x] Given the corrected model, the schema documents the entities and enforces that
      every agency has a Sales Manager (`imobiliarias.gerente_id NOT NULL`), applied
      safely and idempotently.
- [x] The product catalog no longer describes MinhaIncorporadora as a "broker
      assistant".
- [x] `python -m pytest -q` is green with the updated persona assertions.

## Supersedes

- The earlier persona decision captured in commit
  `6282b37 fix: persona = Atendente Comercial (cargo novo), não assistente do gerente`.
  The assistant **is** the Sales Manager's assistant; that commit's framing is
  replaced by this RFC and by
  `decisions/2026-06-25-minhaincorporadora-sales-manager-assistant.md`.
