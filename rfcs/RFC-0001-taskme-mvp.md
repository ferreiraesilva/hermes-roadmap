# RFC-0001: TaskMe MVP

## Status

Draft

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

- Where will tasks be stored?
- How will Hermes identify WhatsApp contacts?
- How will task completion be confirmed?
- How will due date changes be audited?

## Acceptance Criteria

To be defined.