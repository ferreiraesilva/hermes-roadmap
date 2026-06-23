# Decision: Run only Leonardo personal HML for now

## Status

Accepted — 2026-06-23

## Context

MAC02 is the current homologation host for Leonardo's personal Hermes setup. The
legacy host-level Hermes services are still present and have been interfering with
the containerized GitOps-managed deployment.

The `hermes-leonardo-corretores-hml` instance is not needed for the current phase.
The immediate goal is to stabilize the personal TaskMe homologation flow before
running additional Leonardo profiles.

## Decision

For now, Leonardo's HML deployment will keep only the personal profile:

- Active profile: `leonardo/pessoal`.
- Active container: `hermes-leonardo-pessoal-hml`.
- Active product: TaskMe.
- Removed profile: `leonardo/corretores`.
- Removed container target: `hermes-leonardo-corretores-hml`.

The personal HML dashboard may be exposed without authentication because it runs
only inside Leonardo's domestic infrastructure. This exception applies only to the
personal HML environment and must not be copied to client environments or
production.

The dashboard is an integrated Hermes runtime capability, like `hermes chat` or
`hermes gateway`. Any Hermes container can have the dashboard enabled, but that
must be declared explicitly in `hermes-infra` at the profile level. For the
current phase, only `leonardo/pessoal` has the dashboard enabled.

## Consequences

- `hermes-infra` becomes the source of truth for only the Leonardo personal HML
  profile.
- `hermes-leonardo-corretores-hml` must be stopped and removed from MAC02 during
  the cutover.
- The legacy host-level Hermes services must also be stopped or removed so they
  do not compete with the GitOps-managed personal container.
- Dashboard enablement does not create a second Hermes container. It enables the
  dashboard service inside the selected Hermes container.
- WhatsApp pairing and QR-code scanning should happen from the
  `hermes-leonardo-pessoal-hml` container.
- MinhaIncorporadora for Leonardo can be reintroduced later as a new explicit
  roadmap decision or implementation task.

## Related Repositories

- `hermes-infra` — deployment inventory, compose files and cutover scripts.
- `hermes-taskme` — personal TaskMe homologation product.
- `hermes-minhaincorporadora` — parked for Leonardo until the corretores profile
  is reintroduced.
