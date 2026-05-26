# ADR-0001 — Adopt auto-harness as a governance submodule

**Status:** Accepted
**Date:** 2026-05-25
**Deciders:** @unclenate (RealActivity)
**Related:** `HARNESS.md`, `AGENTS.md`, `CLAUDE.md`, `harness.manifest.yaml`, `docs/operating-principles.md`, `docs/knowledge/harness-coverage-gap-analysis.md`

---

## Context

Tula is an open-source personal-health-agent skill pack with companion web
UIs, a Microsoft Waza evaluation gate, and a single-user self-hosted reference
deployment. As the skill set, the apps, and the deployment tooling grew, the
project needed a governance layer that is:

- **Cross-agent** — the repo is worked by Claude Code and, per `AGENTS.md`, by
  Cursor / Windsurf / Copilot / Codex.
- **Auditable** — PHI handling, "draft-never-send" clinical-communication
  invariants, and the eval-before-deploy gate are governance commitments, not
  just conventions.
- **Reusable, not bespoke** — reinventing module/validator/companion-rule
  machinery per project is wasteful.

[auto-harness](https://github.com/unclenate/auto-harness) provides exactly
this: a composable module catalog, path-based companion rules, and a validator
chain, consumed by a project-local `harness.manifest.yaml`.

A first-pass brownfield assessment (see
`docs/knowledge/harness-coverage-gap-analysis.md`) found the catalog did not
yet have vocabulary for Tula's *delivery topology* (an authored, eval-gated
skill pack; a self-hosted-OSS posture). Those gaps were promoted upstream and
landed as the v0.5.2 catalog modules (`architectures/agent-skill-pack`,
`management/eval-gated-testing`, `delivery/self-hosted-oss`), which this
project now activates.

## Decision

Adopt auto-harness as a **git submodule** mounted at `.harness/`, and govern
Tula through a project-local `harness.manifest.yaml` that composes:

- `core/kernel/base`
- `stacks/node-typescript`
- `architectures/web-app` + `architectures/agent-skill-pack`
- `delivery/self-hosted-oss`
- `management/product-lite` + `management/eval-gated-testing`
- `agents/base` + `agents/claude-code` + `agents/generic-llm`

The submodule is pinned to a reviewed commit and updated deliberately via
`git submodule update --remote .harness` (see `HARNESS.md`). Governance rules,
validators, and skills are sourced from `.harness/platform/`.

### Why a submodule (not vendoring or a package dependency)

- **Pinned + reviewable.** The consumer controls exactly which harness commit
  is active; upgrades are explicit diffs, not silent.
- **No runtime coupling.** The harness governs the repo at author/PR/CI time;
  it ships nothing into the OpenClaw runtime or the deployed skill pack.
- **Upstream contribution path.** Working from a submodule made the
  bidirectional first-pass → promote-back → second-pass workflow natural:
  catalog gaps Tula surfaced were contributed upstream before this manifest
  was finalized.

## Consequences

**Positive**

- Tula's governance commitments (PHI containment, synthetic eval fixtures,
  draft-not-send, eval-before-deploy) are expressible as module review gates
  and companion rules rather than tribal knowledge.
- The manifest documents what Tula *is* (eval-gated skill pack, self-hosted
  OSS) precisely, because the catalog gained the matching modules.

**Costs / obligations**

- `required-artifacts` validation is disabled until the module artifacts are
  created (see the manifest's Phase-2 list). Until then the manifest validates
  structurally (manifest + module-graph) but does not enforce artifact
  presence.
- Submodule upgrades require review of the harness diff before re-pinning.
- Governance-entrypoint changes (`HARNESS.md`, `AGENTS.md`, `CLAUDE.md`, CI,
  scripts) now carry a companion-rule expectation: pair them with an ADR or an
  `docs/operating-principles.md` update.

## Alternatives considered

- **Vendor the harness files directly** — rejected: loses the pinned-upstream
  review trail and the clean contribution path.
- **Author bespoke governance** — rejected: duplicates machinery auto-harness
  already provides and would not benefit from upstream improvements.
- **Defer governance until post-1.0** — rejected: PHI handling and the
  eval-before-deploy gate are load-bearing now, not later.

## Notes

This ADR resolves the previously-dangling reference in `AGENTS.md`'s
harness-managed section (which pointed at a submodule-integration ADR that did
not yet exist). The reference now points here.
