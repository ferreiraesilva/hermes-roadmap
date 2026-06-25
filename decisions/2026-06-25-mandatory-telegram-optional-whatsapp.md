# Decision: Telegram bot is mandatory, WhatsApp is optional

## Status

Accepted — 2026-06-25

## Context

All Hermes instances need a reliable communication channel for testing, interaction, and production usage. Telegram bots are free to create, use the official API, do not expire, and do not suffer from the connection instabilities or costs associated with unofficial or bridged channels like WhatsApp. However, certain projects or clients specifically require WhatsApp for user interactions.

## Decision

1. Every Hermes profile/container **must** have a dedicated Telegram bot configured. This ensures a baseline, stable control and communication channel for every instance.
2. WhatsApp integration is **optional**. It can be enabled on a per-project/profile basis in the inventory configuration.
3. For instances where WhatsApp is enabled, the Telegram bot remains active as a primary or fallback interface (e.g., for administrative tasks, direct agent control, or testing).

## Consequences

- The inventory validator in `hermes-infra` will continue to enforce the presence of a Telegram bot token (`telegram_secret` and `telegram_bot_username`) for every profile.
- Enabling WhatsApp (`whatsapp.enabled=true`) will provision the WhatsApp bridge within the container, while keeping the Telegram polling engine active.
- Developers and users can always interact with any Hermes agent via its Telegram bot, even if the WhatsApp bridge is temporarily disconnected or disabled.

## Related

- `decisions/2026-06-21-hermes-naming-standard.md` (Telegram bot naming conventions)
- `hermes-infra` (inventory and client deployment configurations)
