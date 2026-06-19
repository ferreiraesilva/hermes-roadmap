# Product Catalog

This file is the single source of truth for product names used across the Hermes ecosystem.

AI agents must use the product names defined here when filling the `Product` field in initiatives, epics, user stories, technical tasks, spikes, bugs, RFCs and decision records.

Do not create product name variations inside records.

If a new product, module or experiment is needed, the AI agent must propose an update to this catalog before using the new product name as an official value.

## Product Values

### Hermes Core

Use for changes that belong directly to the Hermes base platform.

Examples:

- Core agent behavior
- Message processing
- Base integrations
- Shared Hermes capabilities
- Platform-level configuration

### TaskMe

Use for the Hermes-based task control product.

Examples:

- Task creation
- Task assignment
- Task reminders
- Due date rescheduling
- Task follow-up workflows

### WhatsApp Group Personality

Use for the Hermes module or product that allows each WhatsApp group to have its own behavior, tone, rules and contextual instructions.

Examples:

- Group-specific personality
- Group tone configuration
- Group behavior rules
- Group-level response instructions

### Hermes Experiments

Use for ideas, prototypes or validations that are not yet mature enough to become an official product or module.

Examples:

- Proofs of concept
- Early experiments
- Technical validations
- Uncommitted ideas

### Other

Use only when no existing product value applies.

When using `Other`, the AI agent must add an open question or recommendation asking whether a new product should be added to this catalog.

## Product Naming Rules

- Use exactly the product names listed in this file.
- Do not abbreviate product names.
- Do not create synonyms.
- Do not create variations such as `Hermes`, `WhatsApp Personality`, `Group Personality` or `Task Manager`.
- Prefer updating this catalog over spreading new product names across templates, issues or documentation.
