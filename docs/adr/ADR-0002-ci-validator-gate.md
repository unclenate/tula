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

- **`validate-doc-references`** — *(Originally excluded; **re-included
  2026-05-26** after the upstream fix — see Update below.)* It had hard-required
  a `<project-root>/platform/` directory and exited `2` otherwise, so for a
  *submodule* consumer (platform at `.harness/platform/`) it could not run
  against Tula's own docs.
- **`validate-catalog-counts`** — its recipes and assertions are specific to
  the auto-harness repository structure; not applicable to a consumer. *(Still
  excluded.)*

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

The exclusion of `validate-doc-references` surfaced a likely upstream
inconsistency: `platform/templates/ci/github-actions.yml` included a
doc-references step that exited `2` for the recommended submodule layout, while
`platform/workflow/ci-integration.md`'s minimal workflow omitted it.

## Update (2026-05-26) — doc-references re-included

That upstream defect was filed as auto-harness **OPP-0023** and fixed in
**PRD-0012** (auto-harness #65): `validate-doc-references` no longer requires a
top-level `platform/` — it scans the consumer's own docs (its general
link-resolution pass) and reserves exit `2` for a genuinely missing project
root. The fix is picked up here by bumping the `.harness` submodule.

With the fix landed, `validate-doc-references` is **now included** in this
workflow. Tula's docs pass it after (a) fixing real broken links —
`README.md`'s Discussions link and two `docs/*` "open an issue" links now use
absolute GitHub URLs — and (b) adding a root `.doc-reference-ignore` that
exempts GitHub-valid-but-GitBook-fragile directory targets (e.g. `skills/*/`,
`evals/`), the CI-generated `docs/evals.md`, and the canonical `articles/`
links (maintainer-authored, upstream's to maintain). Tula renders on GitHub,
not GitBook, so the validator's GitBook-fragility class is exempted by design.
`validate-catalog-counts` remains excluded (harness-self specific).
