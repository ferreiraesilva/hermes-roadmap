# RFC-0004: Deploy-managed home SOUL from a product template

## Status

Accepted — implemented in `hermes-infra` (hml) and `hermes-minhaincorporadora`

## Product

Hermes Core

## Summary

The home-level `SOUL.md` of a Hermes container (the agent's native identity at
`/opt/data/SOUL.md`) is currently placed and edited **by hand on the host**. The
deploy does not manage it. This is fragile and drifts: the EBM homologation
container (`hermes-ebm-corretores-hml`) was found running a hand-edited SOUL with
the old, incorrect "Atendente Comercial" persona, while the product repository had
already moved to the corrected "Assistente do Gerente de Vendas" model (RFC-0003).

This RFC makes the home SOUL a **deploy-generated artifact**: a product may ship a
`SOUL.md` template; the deploy renders it with the tenant display name from the
inventory and writes it to the container home. No more hand-editing on the host.

## Motivation

- **Drift**: a hand-edited host file silently diverges from the versioned product.
  The persona the customer actually sees is whatever someone last typed on the box.
- **Loss**: the deploy clones the product with `git reset --hard`, so the plugin is
  always in sync — but the home SOUL is outside that and never updated. The two
  sources of persona (the runtime hook injection in `persona.py` and the home
  `SOUL.md`) can disagree.
- **Per-tenant personalization had no source of truth**: "EBM" only existed in the
  hand-edited SOUL; it never flowed from the inventory. The tenant display name
  should come from `hermes-infra`, the deployment source of truth.
- **GitOps direction**: production deploys are GitOps-only and the host must not be
  edited by hand (see `decisions/2026-06-23-personal-hml-only.md` and the prod
  GitOps plan). A hand-maintained SOUL contradicts that.

## Scope

- A product catalog entry in `hermes-infra` may declare an optional `soul` field: a
  path (relative to the product repo) to a SOUL template.
- The inventory `plan` exposes the resolved **tenant display name** (`tenant_name`,
  defaulting to the client's display name, overridable per profile) and the per
  product `soul` path.
- `deploy-instance.sh` renders the template into `<data_dir>/SOUL.md`
  (`/opt/data/SOUL.md` in the container), substituting the `{{INCORPORADORA}}`
  token with the tenant display name. It backs up any pre-existing unmanaged SOUL
  once (`SOUL.md.pre-managed.bak`) before the first managed write.
- The deploy also writes a generic `HERMES_TENANT_NAME` into each product `.env`, so
  the runtime persona injection can personalize consistently with the SOUL.
- `hermes-minhaincorporadora` turns its `SOUL.md` into the template (token
  `{{INCORPORADORA}}`) and reads `HERMES_TENANT_NAME` as a fallback for
  `MINHAINCORP_INCORPORADORA`.

## Out of Scope

- Rewriting the persona copy itself. The current SOUL text is a placeholder and is
  expected to be refined by the product owner; this RFC only changes how it is
  delivered, not its wording.
- Productionizing the tenant **slug** (`MINHAINCORP_TENANT`, currently the dev
  placeholder `modelo` in the shared catalog). Making the tenant slug per-client is
  a separate follow-up (it ties to which `incorporadoras` row is the tenant and to
  seed/data alignment). See Open Questions.
- Touching the EBM container's current SOUL now. It will be replaced by the managed
  render on the next deploy.

## User Flow

1. Operator runs `deploy-instance.sh <env> <client> <profile>`.
2. The deploy clones the product(s) into `product-src/` as today.
3. If exactly one product in the profile declares a `soul`, the deploy renders it
   with the tenant display name and writes `/opt/data/SOUL.md` (backing up any
   pre-existing unmanaged file once).
4. The container boots with a versioned, tenant-personalized identity that matches
   the product repository.

## Functional Requirements

- A product without a `soul` field leaves the home SOUL untouched (no clobber).
- The render substitutes `{{INCORPORADORA}}` with `tenant_name`; when unset it falls
  back to `incorporadora` so the prose stays grammatical ("da incorporadora").
- A pre-existing unmanaged SOUL is backed up exactly once before the first managed
  write, so a hand-edit is never silently lost.
- The runtime persona injection and the home SOUL use the same tenant display name.

## Non-Functional Requirements

- **Safety**: backup-before-overwrite; never clobber when no template exists; the
  render is idempotent.
- **Genericity**: the deploy stays product-agnostic — it knows only "a product may
  provide a SOUL template" and "the tenant has a display name". No product-specific
  branching in the deploy.
- **Reliability**: more than one SOUL-bearing product in a single profile is a
  defined, non-fatal condition (warn and skip) rather than an ambiguous overwrite.

## Dependencies

- `hermes-infra` — catalog, inventory and `deploy-instance.sh`.
- `hermes-minhaincorporadora` — provides the SOUL template and reads
  `HERMES_TENANT_NAME`.

## Risks

- A profile hosting two SOUL-bearing products has no defined winner. Mitigation: the
  deploy warns and skips; a future `soul_product` profile override resolves it.
- The first managed deploy overwrites the hand-edited EBM SOUL. Mitigation: the old
  file is preserved as `SOUL.md.pre-managed.bak`; and the managed render is the
  corrected, intended persona anyway.

## Open Questions

- Should the tenant **slug** (`MINHAINCORP_TENANT`) become a per-client inventory
  value instead of the shared catalog placeholder `modelo`? Likely yes; deferred to
  a follow-up because it affects tenant-row resolution and seeds.
- For a multi-product profile, how is the SOUL-owning product chosen — first
  declared, or an explicit `soul_product` in the profile? Deferred until a real
  multi-product-with-soul profile exists.

## Acceptance Criteria

- [x] Given a product declares `soul` and the profile has exactly one such product,
      when the deploy runs, then `/opt/data/SOUL.md` is written from the template
      with `{{INCORPORADORA}}` replaced by the tenant display name.
- [x] Given a pre-existing unmanaged SOUL, when the deploy first manages it, then the
      original is preserved as `SOUL.md.pre-managed.bak`.
- [x] Given no product declares `soul`, when the deploy runs, then the home SOUL is
      left untouched.
- [x] Given the rendered tenant name, the runtime persona injection
      (`MINHAINCORP_INCORPORADORA` via `HERMES_TENANT_NAME`) uses the same name.
- [x] `python3 scripts/validate_inventory.py` stays green.

## Related

- `rfcs/RFC-0003-minhaincorporadora-persona-model-correction.md` — the corrected
  persona that the managed SOUL delivers.
- `decisions/2026-06-25-deploy-managed-home-soul.md`.
