# Hermes Base Platform

## Purpose

Hermes Base is the reusable executable foundation for all Hermes Agent installations. It turns approved baseline intent into configuration contracts, schemas, bootstrap guidance, automation contracts and common operating rules.

## Scope

Hermes Base owns:

- Machine-readable baseline and installation defaults.
- Validation schemas for baseline, installation and deployment records.
- Reusable bootstrap and installation lifecycle rules.
- GitHub Project defaults and automation contracts.
- Shared agent operating rules and reusable infrastructure.

Hermes Base does not own product strategy, roadmap governance, deployment decisions or product-specific implementation.

## Repository Relationships

`hermes-roadmap` defines approved intent and tracks why changes are needed. `hermes-base` implements reusable foundation contracts derived from that intent. The roadmap remains the source of truth for official product names and deployment decisions.

Product repositories implement product-specific behavior. Hermes Base may define generic extension contracts, but it must not contain TaskMe, Multichannel, WhatsApp Group Personality, Investments or other product logic.

## Change Governance

Baseline behavior and deployment contract changes require human approval. Agents may draft and validate changes but must not approve sprint execution, merge pull requests, activate deployments or close issues without human acceptance.

## Initial Direction

The initial repository should remain intentionally small: configuration contracts, JSON schemas, bootstrap documentation, templates and agent rules. Runtime code should be added only when an approved reusable requirement makes it necessary.
