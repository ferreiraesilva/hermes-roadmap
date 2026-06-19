# Hermes Ecosystem Architecture

## Purpose

The Hermes Ecosystem separates planning, shared executable foundations and product implementation so agents can place work in the correct repository.

## Repository Responsibilities

### `hermes-roadmap`

The planning and governance layer. It records product intent, roadmap items, issues, RFCs, decisions, the product catalog, installation intent and product deployment decisions. It defines what should exist and why.

### `hermes-base`

The executable shared foundation for every Hermes Agent installation. It owns reusable baseline configuration, schemas, bootstrap contracts, GitHub automation contracts, common operating rules and shared infrastructure. It defines how installations are created, configured and operated.

### Product Repositories

Product repositories own product-specific behavior and delivery. Product logic must not be implemented in `hermes-roadmap` or `hermes-base`.

## Three-Layer Runtime Model

1. **Hermes Base:** inherited shared behavior and contracts.
2. **Hermes Installation:** a concrete agent instance with identity, context, permissions and enabled products.
3. **Product / Module Deployment:** an explicit decision that enables a product in all, selected or no installations.

Every Hermes installation should inherit Hermes Base unless the human explicitly approves an exception.

A product being present in the product catalog does not mean it is available in every Hermes installation.

## Ownership Flow

Architecture intent and deployment decisions begin in `hermes-roadmap`. Approved reusable contracts are implemented in `hermes-base`. Approved product behavior is implemented in the relevant product repository. Human approval remains required for sprint commitment, baseline changes, deployment changes, pull request merge and item closure.

## Placement Examples

- A proposed baseline rule belongs in `hermes-roadmap`; its approved executable contract belongs in `hermes-base`.
- Installation intent is planned in `hermes-roadmap`; reusable provisioning and validation belong in `hermes-base`.
- Task assignment logic belongs in the TaskMe repository.
- A product catalog entry does not activate deployment anywhere.
