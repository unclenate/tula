# Operating Principles

How the Tula project is **built, governed, and evolved**. These are
engineering and governance principles for contributors and AI agents working
*on* the repository. They are distinct from — and sit alongside — the
product / clinical / ethical principles in [`principles.md`](principles.md),
which describe what Tula does *for users*. When the two are both relevant, the
product principles set the goal and these set how the work is done.

**Owner:** @unclenate (RealActivity) | **Last Updated:** 2026-05-25 | **Review Cycle:** On-change

---

## 1. Governance is sourced from auto-harness

Tula is governed by the modules declared in `harness.manifest.yaml`, with
auto-harness mounted at `.harness/`. Governance rules, validators, and skills
come from there; project-specific decisions are recorded as ADRs under
`docs/adr/`. See [ADR-0001](adr/ADR-0001-submodule-integration.md) for the
adoption rationale. Changing a governance entrypoint (`HARNESS.md`,
`AGENTS.md`, `CLAUDE.md`, CI workflows, `scripts/`) is a substantive act:
pair it with an ADR or an update to this file in the same change.

## 2. PHI never enters the repository

Personal health information stays in the user's runtime workspace, never in
version control. Skills **reference** personal data resolved at runtime; they
never **embed** it (see [`skills/AGENTS.md`](../skills/AGENTS.md) §
"Personal Data: Reference, Don't Embed"). Evaluation fixtures use synthetic
personas only; the de-identification helper
(`skills/request-amendment/scripts/redact_phi_for_eval.mjs`) exists to keep
that line bright. A change that would place real PHI under version control is a
governance failure, not a convenience.

## 3. Skills are scope-contained, least-permission, and eval-gated

Each skill does one job and writes only to its own declared workspace cache.
A new or changed skill ships with a matching eval (or, failing that, an
authoring-conventions/ADR update) — an unevaluated skill is an ungoverned
capability loaded into the agent runtime. This is the
`architectures/agent-skill-pack` contract; the reference shape is
`skills/med-pdf/` and the authoring standard is `skills/AGENTS.md`.

## 4. Quality is gated on evaluations, not coverage

Tula's merge gate is binary-graded evaluation of skill behavior via Microsoft
Waza, not line/branch coverage (`management/eval-gated-testing`). Every skill
carries `evals/<skill>/` with task fixtures, graders, and at least a
`basic-usage` and a `should-not-trigger` case. Thresholds (e.g.
`task_completion ≥ 0.8`) are quality commitments: an agent must not lower a
threshold or remove an anti-trigger case without human approval. Status is
regenerated to `docs/evals.md`.

## 5. Consequential actions are drafted, never auto-executed

Clinical communications (portal messages, amendment requests) are produced as
**drafts a human sends** — the agent never transmits to a provider. Triage
red-flag detection runs *before* any draft and can pre-empt it with an
emergency redirect. These human-in-the-loop checkpoints are invariants: a code
path that auto-confirms a consequential action is a governance failure, not a
UX shortcut.

## 6. The model-provider seam stays open

Tula routes each task to the most capable, cost-effective, and
privacy-appropriate model available in the deployment, across providers and
foundries (see [`model-routing.md`](model-routing.md)). No skill hard-codes a
single provider; routing is deployment-context-aware. This keeps air-gapped
open-weight deployments first-class, not an afterthought.

## 7. Self-hosted OSS is the delivery posture

Tula ships as published Apache-2.0 software the **user operates** on their own
infrastructure (`delivery/self-hosted-oss`) — not a hosted multi-tenant
service. The operator inherits the security posture the project ships, so
changes to install/deploy/backup behavior are documented in the self-hosting
guide and surfaced honestly (default credentials, exposed ports, data
locations). The hosted, multi-tenant story is a separate product (Aria) with
its own repository and manifest, per [`OPEN_CORE.md`](../OPEN_CORE.md).

## 8. Prefer text, keep the trace shared

Skills, configs, evals, and docs are text — versionable, diffable,
toolchain-free. The trace surface is OpenTelemetry-shaped so any compatible
collector can consume it. Institutional knowledge lives in the git-tracked,
shareable store (docs, ADRs), not in an agent's private memory, which is a
cache rather than the canonical record.
