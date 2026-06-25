# Decision: Automatically update the HML Hermes image

## Status

Accepted.

## Context

The MAC02 HML Hermes deployment has been drifting behind the current Hermes
container image. The deployment intentionally pins images by digest for
reproducibility, but that means a separate mechanism is required to discover and
apply newer upstream images.

The current active personal HML deployment is `hml/leonardo/pessoal`.

## Decision

`hermes-infra` will own the daily HML image update flow.

The update flow will:

- check the configured Hermes image tag daily;
- resolve the tag to an immutable digest;
- write the selected digest to the environment runtime state;
- redeploy the target profile through the normal inventory-driven deploy script;
- run a smoke test after deploy;
- roll back to the previous image if deploy or smoke validation fails.

For the current phase, the scheduled update applies only to
`hml/leonardo/pessoal`.

The native `hermes update` command is intentionally not used for this Docker
deployment. In the published container image, `hermes update --check` reports
that the command does not apply inside Docker and instructs operators to pull a
fresh image and restart/recreate the container. The internal Hermes cron is also
not used for container upgrades because it would require exposing host Docker
control inside the application container and would depend on the container being
healthy before it can repair or replace itself.

The scheduled update also compares the Docker image revision label with
`refs/heads/main` from `https://github.com/NousResearch/Hermes-Agent.git`. HML
runs in strict mode, so if the published Docker image lags behind upstream Git,
the update fails visibly instead of silently accepting a stale image.

## Consequences

- HML stays current without manually recreating the container.
- The running image remains auditable because the applied value is a digest, not
  only a mutable tag.
- The update path exercises the same deploy flow used for manual recovery.
- Product data, auth files and WhatsApp session state remain in the profile home
  and are not recreated by the image update.
- New Hermes images require an auth provider for non-loopback dashboards; the
  personal HML dashboard uses simple basic auth supplied from `hermes-infra`
  secrets.
- Client/prod automatic updates remain out of scope until their rollback and
  approval policies are defined.

## Implementation Notes

The implementation lives in `hermes-infra`:

- `scripts/update-hermes-image.sh`
- `scripts/install-hermes-update-timer.sh`
- `runtime_root/hermes-image.env` on the target host

The MAC02 timer is a `systemd --user` timer so the scheduled command stays close
to the normal user-level deployment model.
