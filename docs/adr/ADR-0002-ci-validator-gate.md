# ADR-0002 — Gate Tula on the harness validator chain in CI

**Status:** Accepted
**Date:** 2026-05-25
**Deciders:** @unclenate (RealActivity)
**Related:** [`ADR-0001`](ADR-0001-submodule-integration.md), `harness.manifest.yaml`, `.github/workflows/harness.yml`, `.harness/platform/workflow/ci-integration.md`

---

## Context

ADR-0001 adopted auto-harness and the second-pass intake reached full local
compliance (`required-artifacts` enabled; manifest, module-graph,
required-artifacts, placeholders, and agent-pack all green). Green *locally* is
not green *enforced* — without CI, governance drift (a deleted required
artifact, an unfilled template token, a companion-rule violation) only surfaces
if someone runs the validators by hand.

Tula already has one workflow, `.github/workflows/eval-status.yml`, which runs
the Microsoft Waza skill-eval gate and regenerates `docs/evals.md`. That gate
covers *skill behavior*. It does not cover *governance structure*.

## Decision

Add `.github/workflows/harness.yml` running the auto-harness **consumer
validator chain** on every pull request and push to `main`:

- `validate-manifest` — manifest schema and project fields
- `validate-module-graph` — module dependencies, conflicts, types
- `validate-required-artifacts` — every required file exists
- `validate-placeholders` — no unfilled `[[…]]` / date tokens (honors
  `.placeholder-ignore`)
- `validate-agent-pack` — agent entrypoints present and consistent
- `validate-companions` — PR-diff satisfies companion rules (pull_request only)

The job checks out the `.harness` submodule (`submodules: recursive`) with full
history (`fetch-depth: 0`, required by the companion validator), installs Ruby
3.3 and ripgrep, and runs the chain from `.harness/platform/validators/`. It
coexists with `eval-status.yml`; the two gates are complementary (governance
structure vs. skill behavior).

### Validators deliberately excluded

- **`validate-doc-references`** — hard-requires a `<project-root>/platform/`
  directory (it scans the harness's own platform docs) and exits `2` (usage
  error) otherwise. For a *submodule* consumer the platform lives at
  `.harness/platform/`, so this validator cannot run against Tula's own docs.
  It is therefore omitted, matching the `ci-integration.md` "Minimal Working
  Workflow" (which omits it) rather than the `templates/ci/github-actions.yml`
  starter (which includes it — a mismatch worth fixing upstream; see Notes).
- **`validate-catalog-counts`** — its recipes and assertions are specific to
  the auto-harness repository structure; not applicable to a consumer.

## Consequences

**Positive**

- Governance compliance is enforced on every PR/push, not just when someone
  remembers to run validators. A merge that breaks the manifest, drops a
  required artifact, or violates a companion rule is caught automatically.
- The companion-rule gate now actively protects governance entrypoints
  (`HARNESS.md`, `AGENTS.md`, `CLAUDE.md`, `.github/workflows/`, `scripts/`) —
  changing one without a paired ADR / operating-principles / change-log entry
  fails CI. (This ADR is itself the satisfier for adding the workflow.)

**Costs**

- Two CI workflows now run on Tula PRs (eval-status + harness validators);
  modest added minutes.
- Submodule upgrades that change validator behavior can change CI outcomes;
  pin and review per ADR-0001.

## Notes

The exclusion of `validate-doc-references` surfaces a likely upstream
inconsistency: `platform/templates/ci/github-actions.yml` includes a
doc-references step that exits `2` for the recommended submodule layout, while
`platform/workflow/ci-integration.md`'s minimal workflow omits it. A future
upstream fix would make the validator either no-op or scan consumer docs when
no `<root>/platform/` exists. Recorded here for a potential auto-harness
observation/OPP; not actioned from this consumer repo.
