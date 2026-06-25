# Decision: Preserve Hermes home ownership in Docker deployments

## Status

Accepted.

## Context

Hermes Docker profiles persist their runtime state in `/opt/data`, backed by a
host bind mount. Operational commands executed with plain `docker exec` run as
`root` by default. When those commands create files under `/opt/data`, the
resulting `root:root` ownership can break later gateway, dashboard, setup and
WhatsApp flows.

This issue has recurred during WhatsApp pairing and dashboard/gateway restarts.

## Decision

`hermes-infra` treats `/opt/data` ownership as an explicit operational invariant.

All manual Hermes commands inside a profile container must use a versioned
wrapper that executes as the host deploy user UID/GID instead of Docker's default
root user.

`hermes-infra` also provides a repair command to normalize ownership when a
manual command or one-off diagnostic already created `root:root` files.

## Consequences

- Operators must not run `docker exec <container> hermes ...` directly for
  Hermes commands that may write to `/opt/data`.
- WhatsApp pairing must use the safe wrapper so session files are writable by
  the gateway after restart.
- Deploy continues to normalize ownership defensively, but manual commands
  between deploys still need the wrapper.
- The recurring `root:root` failure mode is documented and has an explicit
  remediation path.

## Implementation Notes

Implemented in `hermes-infra`:

- `scripts/hermes-command.sh`
- `scripts/repair-instance-permissions.sh`
- README runbook section for persistent home ownership
