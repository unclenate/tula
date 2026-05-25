# Harness Coverage Gap Analysis — Tula

**Owner:** @unclenate
**Created:** 2026-05-24
**Last Updated:** 2026-05-24
**Consumer repo:** `tula` (this repo)
**Harness:** auto-harness, mounted at `.harness/` (separate repo:
`github.com/unclenate/auto-harness`, branch `main` @ `7970a6e`)

---

## Purpose

This document records where the auto-harness module catalog **could** and
**could not** structurally describe Tula during the 2026-05-24 brownfield
onboarding first pass. It is the consumer-side evidence artifact: the
generalized version of each gap below is promoted to an `OPP-NNNN` in the
auto-harness repo, which cites this file by gap number (`§TG1` … `§TG5`).

Gaps use a `TG` (Tula Gap) prefix to avoid collision with the `G#` numbering
in OpenEMR's own gap analysis, since both are cited from shared auto-harness
OPPs.

**Scope discipline.** This file describes *Tula's* experience of the catalog.
It stays in the consumer repo. Anything that generalizes beyond Tula (the
reusable module/template/skill shape) belongs in an auto-harness OPP, not
here. Tula-specific implementation detail never travels into auto-harness.

---

## Methodology context

This is the **third** external brownfield onboarding in the 2026-05-24
cycle, after YouBase (Node + CoffeeScript cryptographic identity store →
OPP-0008/0009/0010) and OpenEMR (25-year-old PHP healthcare EHR →
OPP-0011…0017).

The first two passes converged on **stack/data catalog *breadth*** (a
Node-not-TypeScript stack, a PHP stack, embedded key-value, relational-SQL
generalization). Tula surfaces a **different gap class**: **delivery-topology
breadth for *agent-native* products**. Tula is not under-described because
its *language* is exotic (it is plain Node/TypeScript, which the catalog
handles) — it is under-described because the *unit of product* is an
**authored, eval-gated skill pack running on an agent runtime**, and the
catalog has no vocabulary for that shape.

This matters for the second pass: you cannot profile Tula's app accurately
against a model that cannot name how Tula is built and shipped. The first
pass closes that vocabulary gap; the second pass measures Tula with it.

---

## What auto-harness described well (no gap)

The catalog covered the conventional layers of Tula cleanly. These are
**not** gaps; recording them keeps the assessment honest and bounds the
gap set to what is genuinely missing.

| Layer | Module that fits | Evidence in Tula |
|---|---|---|
| Stack | `stacks/node-typescript` | `apps/*/package.json` + `tsconfig.json`; skill scripts are Node ESM `.mjs` |
| Architecture (surface apps) | `architectures/web-app` | `apps/agent-studio/`, `apps/my-aria/` — Next.js 15 App Router |
| Product framing | `management/product-lite` | `README.md`, `OPEN_CORE.md`, `docs/use-cases.md` |
| Operating principles | kernel `docs/operating-principles.md` | `docs/principles.md` (EQUIVALENT — 10 filled principles) |
| Dev-agent contract | `agents/{base, claude-code, generic-llm}` | `AGENTS.md`, `CLAUDE.md`, `.claude/`, cross-client declaration |
| Agent-workspace files | `agents/openclaw` (partial) | `docs/agent/{SOUL,IDENTITY,HEARTBEAT,AGENTS}.md`, `TOOLS.md.example` |

The catalog's miss is narrow and specific: the **agent-native delivery
topology** and **patient-side health safety**. Everything below is one of
those two themes.

---

## Gap summary

| # | Gap | Auto-harness today | Disposition |
|---|---|---|---|
| TG1 | Eval-gated authored **skill pack** as a buildable/shippable shape | `agents/openclaw` (workspace files only); OPP-0001/0002 adjacent but distinct | **New OPP** |
| TG2 | **Binary-LLM-eval** quality gate as a testing posture | `management/testing-standard` (%-coverage); OPP-0015 (external regulator kits) | **New OPP** |
| TG3 | **`delivery/self-hosted-oss`** posture | `delivery/{prototype, production-saas, internal-platform}` | **New OPP** |
| TG4 | **Patient-facing health-agent safety** | OPP-0013 healthcare family (operator/server-side) | **New OPP** |
| TG5 | `healthcare-fhir` / `healthcare-smart-on-fhir` from the **patient-authorized-client** role | OPP-0013/0016 (grounded in OpenEMR's server/provider-launch role) | **Augment OPP-0013/0016** |
| TG6 | Multi-provider model-routing seam | none observed | **Watch** (thesis-only) |

---

## TG1 — Eval-gated authored skill pack (delivery/architecture)

**What it is in Tula.** Tula's *product* is a collection of authored agent
skills, not an application binary or a service. Each skill is a governed
unit: `skills/<skill>/SKILL.md` (the spec) + `references/` (progressive
disclosure) + `scripts/*.mjs` (deterministic tools). Six live skills:
`health-records`, `med-pdf`, `epic-note`, `myhealth-pulse`, `memory-diff`,
`request-amendment`. Authoring is governed by `skills/AGENTS.md` — a mature
standard covering frontmatter spec, body-section order, the
"reference-don't-embed personal data" rule, token discipline, and a
deploy-after-validation gate. Skills are deployed via
`scripts/deploy-skills.sh` to `~/.openclaw/workspace/skills/` and the
ecosystem distributes community skills through ClawHub.

**Code-path anchors:** `skills/AGENTS.md`, `skills/med-pdf/` (reference
template), `skills/*/SKILL.md`, `skills/*/references/`, `skills/*/scripts/*.mjs`,
`scripts/deploy-skills.sh`, `scripts/set-openclaw-health-skillz-env.sh`.

**What auto-harness has today, and why none fits.**
- `agents/openclaw` governs the *workspace files* the runtime reads
  (`TOOLS.md`, `SOUL.md`, `HEARTBEAT.md`) — but not the *production model*
  for authoring and shipping a conventioned skill collection.
- `architectures/agentic-ui` / `domains/agentic-interfaces` govern
  *in-product* agent UIs (copilots, generative UI) — Tula's skills are not
  an in-product UI; they are portable capabilities loaded by a runtime.
- `architectures/mcp-server` governs an MCP server's exposed tool surface —
  Tula ships no MCP server.
- OPP-0001 (exportable governance *to* runtimes like Hive/LangGraph) and
  OPP-0002 (in-product agentic interfaces) are the nearest neighbors but
  point the other way: OPP-0001 exports governance to a runtime; Tula is a
  *payload authored for* a runtime. Neither governs "an authored,
  spec-conformant, eval-gated skill pack as the deliverable."

**Proposed shape (for the OPP to take a position on).** A module governing
the authored-skill-pack delivery topology: skill-scoping discipline
(one skill, one job), least-permission workspace-cache boundaries
(`.health-records-cache/`, `.med-pdf-cache/` scoped to the skill),
reference-don't-embed personal data, the SKILL.md spec contract, and
deploy-after-eval-gate. Open question the OPP must resolve: is this an
`architecture/` (topology), a `domain/` (ecosystem: OpenClaw/ClawHub/
agentskills.io), or a `delivery/` overlay? Evidence leans architecture +
a thin OpenClaw-ecosystem domain.

**Disposition:** New OPP. Tula is the first evidence point.

---

## TG2 — Binary-LLM-eval testing posture

**What it is in Tula.** Tula's quality gate is **binary-graded evaluation
of agent skill outputs**, run by Microsoft Waza in CI on every PR touching
skills or evals. Each skill has `evals/<skill>/eval.yaml` (metric +
threshold + graders), a `tasks/` set, and synthetic `fixtures/`. The task
taxonomy encodes intent coverage, not line coverage:
`basic-usage`, `edge-case`, `should-not-trigger` (anti-trigger), and
domain-specific cases like `triage-override`. Graders include `not_empty`
and `under_word_budget`; thresholds like `task_completion ≥ 0.8` gate merge.
A `redact_phi_for_eval.mjs` script enforces the synthetic-fixtures-only rule.

**Code-path anchors:** `.waza.yaml`, `evals/<skill>/eval.yaml`,
`evals/<skill>/tasks/`, `evals/<skill>/fixtures/`,
`.github/workflows/eval-status.yml`, `docs/evals.md` (CI-generated),
`scripts/generate-eval-status.sh`,
`skills/request-amendment/scripts/redact_phi_for_eval.mjs`.

**What auto-harness has today, and why none fits.**
- `management/testing-standard` is **%-coverage shaped**: its
  `coverage-thresholds.md` template expresses unit/integration coverage
  percentages and a testing pyramid. It cannot express grader thresholds,
  the eval task taxonomy, or the synthetic-fixture rule.
- OPP-0015 (regulated-compliance external test kits, e.g. Inferno ONC G10)
  is a *different* pattern: a third-party conformance suite mounted as a
  submodule and invoked by exit code — not LLM-output evaluation authored
  alongside the unit under test.

**Proposed shape.** A `testing-standard` *variant* or sibling overlay for
eval-gated quality: grader-threshold artifact, eval task taxonomy
(basic / edge / should-not-trigger / domain-override), synthetic-fixture
discipline, and spec-gate-in-CI. Mirrors the Microsoft Foundry
"evaluators as unit tests" model the project README already cites.

**Disposition:** New OPP. Pairs with TG1 (the eval gate is how the skill
pack from TG1 earns merge).

---

## TG3 — `delivery/self-hosted-oss` posture

**What it is in Tula.** Tula is a published Apache-2.0 open-source tool
**and** a single-user self-hosted reference deployment. It is operated by
the end user on their own VM; there is no hosted service.

**Code-path anchors:** `OPEN_CORE.md`, `LICENSE`, `docs/deployment-guide.md`,
`docs/cost-guide.md`, `docs/security-model.md`,
`scripts/install-health-skillz-vm.sh`, `scripts/health-skillz-vm-preflight.sh`,
`scripts/agent-backup.sh`, `scripts/deploy-skills.sh`,
`docs/health-skillz-vm-hosting.md`.

**What auto-harness has today, and why none fits.**
- `delivery/prototype` *undersells* Tula: the skills are live, they handle
  real PHI, there is a published security model, and a CI eval gate. This is
  past "experiment."
- `delivery/production-saas` *oversells* Tula: it requires
  `docs/ops/{environment-inventory,release-checklist,rollback-checklist}.md`
  for hosted infrastructure that does not exist — each user self-hosts.
- `delivery/internal-platform` is for internal shared tooling with no
  external surface; Tula is externally published OSS.

The honest consequence today: the manifest must pick a posture that
misrepresents the project. The one production artifact that genuinely
applies regardless — `docs/security/risk-register.md`, given PHI — is
gated behind a `production-saas` framing that otherwise does not fit.

**Proposed shape.** `delivery/self-hosted-oss`: required artifacts =
self-host deployment guide + security model + distributable
versioning/release-intent; **no** hosted-infra ops artifacts. Distinct
from OPP-0015's `regulated-saas.yaml`, which extends `production-saas`.

**Disposition:** New OPP. Tula is the first evidence point.

---

## TG4 — Patient-facing health-agent safety

**What it is in Tula.** Tula governs healthcare from the **patient side**,
which surfaces safety concerns OpenEMR's operator-side family does not:

- **Triage-first red-flag gating** — `epic-note` scans for emergency red
  flags and returns a 911 redirect *before* drafting any message
  (`skills/epic-note/references/triage-rules.md`).
- **Draft-never-send** human-in-the-loop — portal messages and amendment
  requests are drafts; the agent never auto-sends to a provider
  (`skills/epic-note/`, `skills/request-amendment/`).
- **Non-diagnostic stance** — an explicit operating principle, not just a
  disclaimer (`docs/principles.md` § "AI-Assisted Interpretation, Not
  Clinical Diagnosis").
- **PHI at the agent-workspace boundary** — clinical data is confined to
  workspace caches (`.health-records-cache/`, `.med-pdf-cache/`), never
  embedded in skills.
- **Indirect prompt injection via ingestion** — the email router locks
  inbound mail to a sender allowlist at the Exchange transport layer before
  any model sees it; the threat is named honestly
  (`docs/security-model.md`, `docs/email-router-design.md`).
- **Clinical-significance tiering** — `memory-diff` ranks change by clinical
  significance (`skills/memory-diff/references/clinical-significance.md`).

**Code-path anchors:** `skills/epic-note/references/triage-rules.md`,
`skills/request-amendment/`, `skills/memory-diff/references/clinical-significance.md`,
`docs/principles.md`, `docs/security-model.md`, `docs/email-router-design.md`.

**What auto-harness has today, and why it doesn't fit.**
- OPP-0013's `domains/healthcare-*` family is **operator/server-side** —
  FHIR server, HL7v2, CCDA, ePrescribing, audit-log, EHI-export. None of
  its 12 sub-modules govern a *patient-facing agent's* clinical safety.
- `domains/agentic-interfaces` covers prompt-injection / action-approval
  for in-product agent UIs generically, but not health-specific
  patient-safety (triage gating, non-diagnostic stance, draft-not-send
  clinical comms).

**Proposed shape.** A patient-side counterpart to OPP-0013: either a new
`domains/healthcare-patient-agent` sub-module in that family, or a
cross-cutting overlay. Governs triage/red-flag gating, non-diagnostic
disclaimer discipline, draft-not-send for clinical communications,
PHI-workspace-boundary, and indirect-injection-via-ingestion.

**Disposition:** New OPP, explicitly framed as the patient-side complement
to OPP-0013 (which only saw the operator side).

---

## TG5 — `healthcare-fhir` / `healthcare-smart-on-fhir` from the patient-client role (augmentation)

**What it is in Tula.** `skills/health-records` is a **patient-authorized
SMART-on-FHIR client**: the patient grants OAuth access to read their *own*
records from Epic MyChart / Oracle-Cerner, which land as FHIR R4 JSON in the
workspace cache. (Derived from `jmandel/health-skillz`; see `NOTICE`.)

**Code-path anchors:** `skills/health-records/SKILL.md`,
`skills/health-records/scripts/{check-backend,create-session,finalize-session}.mjs`,
`skills/health-records/references/fhir-guide.md`,
`apps/my-aria/lib/` (FHIR types), workspace `.health-records-cache/<date>/<provider>.json`.

**Why this augments rather than re-files.** OPP-0013 already proposes
`domains/healthcare-fhir` and `domains/healthcare-smart-on-fhir`, but both
are grounded in OpenEMR's **server / provider-launch** role. OPP-0013's own
Risks section asks for "re-evaluation after the second healthcare consumer
onboards." Tula **is** that second consumer, and it exercises the same
sub-modules from the **patient-authorized-client / patient-access** role —
a different scope map (`patient/*.read` vs provider launch scopes), a
different trust model (the patient is the resource owner), and a
read-own-records rather than operate-the-server posture. This validates the
sub-module boundary and refines its required-artifact shape (a SMART scope
map that distinguishes patient-access from provider-launch).

**Disposition:** Augment OPP-0013 (and OPP-0016's `harness-fhir` /
specialist-skill scope) with Tula as a second, role-distinct evidence point.
Do **not** file a duplicate healthcare-FHIR OPP.

---

## TG6 — Multi-provider model-routing seam (watch)

**What it is in Tula.** A deployment-context-aware routing seam that directs
each task to the most capable / cost-effective / privacy-appropriate model
available in that deployment (Anthropic / OpenAI / Gemini / vLLM open-weights;
Azure / NVIDIA / Palantir foundries; healthcare models like MedGemma).

**Code-path anchors:** `docs/model-routing.md`, `.waza.yaml` (`defaults.model`,
`dev.model`).

**Why "watch," not a full gap.** This is real but **thesis-only** for now —
a single evidence point, and arguably an architecture concern rather than a
distinct governable shape. No second consumer has exercised it. Recorded so
a future pass can promote it if the pattern recurs; not promoted to an OPP
in this pass.

**Disposition:** Watch. No OPP this cycle.

---

## Meta-observation (for shared-observations on the auto-harness side)

Three brownfield passes in one cycle surfaced **two distinct gap classes**:
YouBase + OpenEMR converged on **stack/data catalog breadth** (language and
storage variety); Tula surfaced **delivery-topology breadth for agent-native
products** (the unit of product is an eval-gated skill pack on a runtime, not
an app or a service). The catalog's conventional layers (stack, web-app,
product, dev-agent) described Tula fine — the miss is concentrated entirely
in *how an agent-native product is built, gated, and shipped*. This suggests
the next catalog-breadth investment after the YouBase/OpenEMR stack work is
**agent-native delivery**, not more language stacks.

---

## Next actions (this pass)

1. **This document** — consumer-side evidence. (done)
2. **Auto-harness OPP batch** (separate repo, `Status: proposed`, citing
   `§TG1…§TG5`): four new OPPs (TG1 skill-pack delivery, TG2 binary-LLM-eval
   testing, TG3 `delivery/self-hosted-oss`, TG4 patient-agent safety) +
   augment OPP-0013/0016 with TG5 patient-client SMART-on-FHIR evidence.
3. **Second pass (brownfield intake)** — profile Tula's app against the
   now-completed structural model: `harness.manifest.yaml`,
   `docs/adr/ADR-0001-submodule-integration.md`,
   `docs/operating-principles.md`, root `.placeholder-ignore`.

---

## Second pass — resolution (2026-05-25)

The auto-harness OPP batch (step 2) merged as PR #53; the three unblocked
agent-native modules then shipped as the **v0.5.2** catalog patch (PR #55:
`architectures/agent-skill-pack`, `management/eval-gated-testing`,
`delivery/self-hosted-oss`). With the catalog enriched, the second-pass intake
ran against it. The headline gaps no longer read "no module fits":

| Gap | First pass | Second pass |
|-----|-----------|-------------|
| TG1 skill-pack delivery | New OPP (no module) | **Activated** `architectures/agent-skill-pack` |
| TG2 binary-eval testing | New OPP (testing-standard mismatch) | **Activated** `management/eval-gated-testing` |
| TG3 self-hosted-oss posture | New OPP (prototype/saas both wrong) | **Activated** `delivery/self-hosted-oss` |
| TG4 patient-agent safety | New OPP (bias-gated) | Deferred — OPP-0022 held behind the US-healthcare-bias guardrail |
| TG5 healthcare-fhir/smart (patient-client) | Augment OPP-0013/0016 | Deferred — bias-gated healthcare family |
| TG6 model-routing seam | Watch | Watch (operating-principles § 6 names it; no module yet) |

**Composition activated** (`harness.manifest.yaml`, id `tula`, maturity `mvp`,
criticality `high`): `core/kernel/base` · `stacks/node-typescript` ·
`architectures/web-app` + `architectures/agent-skill-pack` ·
`delivery/self-hosted-oss` · `management/product-lite` +
`management/eval-gated-testing` · `agents/base` + `claude-code` + `generic-llm`.

**Intake artifacts created:** `harness.manifest.yaml` (real composition),
`docs/adr/ADR-0001-submodule-integration.md` (resolves the previously-dangling
`AGENTS.md` reference), `docs/operating-principles.md` (kernel triad complete),
root `.placeholder-ignore` (clears the dated-memory-path false positives).

**Validators green:** manifest, module-graph, required-artifacts (disabled),
placeholders.

**Phase 2 (to enable `required-artifacts`):** create
`docs/architecture/overview.md`, `docs/testing/eval-strategy.md`,
`docs/deployment/self-hosting-guide.md`,
`docs/product/{problem-statement,requirements,release-intent}.md`,
`.claude/settings.json` — then remove `required-artifacts` from
`disabledValidations` and wire validators to CI.
