# RFC-0001: TaskMe MVP

## Status

Implemented — v1 shipped, in homologation (`feature/taskme-v1` branch, `ferreiraesilva/TaskMe`)

## Summary

TaskMe is a Hermes-based task control system that allows a user to create, assign, remind and follow up on tasks through text or audio messages.

## Motivation

The goal is to use Hermes as an operational assistant for task assignment and follow-up, especially through WhatsApp.

## Initial Scope

- Create a task from a text message
- Create a task from an audio message
- Transcribe audio before processing
- Identify the assigned person
- Validate the task due date
- Send the task to the assigned person
- Send weekly reminders until completion
- Allow due date rescheduling with justification

## Out of Scope

- Full web dashboard
- Complex permission model
- Multi-company support
- Advanced analytics

## Open Questions

~~- Where will tasks be stored?~~ Postgres (local container, managed by `hermes-infra`)
~~- How will Hermes identify WhatsApp contacts?~~ Phone number as canonical identity; multi-channel via `channels` table (`0002_channels.sql`)
~~- How will task completion be confirmed?~~ Assigned person replies; status updated via `taskme_concluir_tarefa` tool
~~- How will due date changes be audited?~~ `reprogram` service logs reason and new date

## Acceptance Criteria

- [x] Task created from text message
- [x] Assigned person identified by phone number
- [x] Task sent to assigned person
- [x] Weekly digest on Mondays
- [x] Daily digest
- [x] Due date rescheduling with justification
- [x] Charge reminders on due date
- [x] Multi-channel delivery (WhatsApp + Telegram)