# Hermes Ecosystem

## Vision

Hermes is the base agent platform used to build automation, communication and productivity products.

The Hermes ecosystem includes modules, features and complete products that depend on Hermes to operate.

## Products and Modules

### Shipped (v1 built, in homologation)

- **TaskMe** — task assignment and follow-up via WhatsApp and Telegram. Multi-channel identity, daily digests, charge reminders.
- **MinhaIncorporadora** — real estate broker assistant. Unit search, pricing, media, human escalation. Multi-tenant per installation.

### Planned

- **WhatsApp Group Personality** — per-group behavior, tone and rules. See RFC-0002.
- **Hermes Experiments** — proofs of concept and early validations not yet mature enough for a product.

## Principles

- Hermes should remain the base platform.
- Small reusable capabilities should be treated as modules.
- Larger initiatives should be treated as products.
- Product and technical decisions should be documented before implementation.
- Roadmap items should be visible, simple and actionable.

## Current Status

Two products shipped and in homologation (TaskMe, MinhaIncorporadora).
Infrastructure managed by `hermes-infra` (inventory-driven multi-tenant deployments).