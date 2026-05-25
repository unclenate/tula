# Requirements

What Tula must do and the constraints it must hold. Required by
`management/product-lite`. Derived from [`../use-cases.md`](../use-cases.md),
[`README.md`](../../README.md), and [`OPEN_CORE.md`](../../OPEN_CORE.md).

**Owner:** @unclenate (RealActivity) | **Last Updated:** 2026-05-25

---

## Functional requirements

| # | Requirement | Status / Skill |
|---|-------------|----------------|
| FR-1 | Extract structured FHIR R4 data from photographed/emailed health documents (labs, prescriptions, EOBs, discharge notes, imaging reports, device readings) | Live — `med-pdf` + email router |
| FR-2 | Pull a patient's own records from portals via patient-authorized SMART on FHIR | Live — `health-records` |
| FR-3 | Draft (never auto-send) patient-portal messages with triage-first red-flag gating | Live — `epic-note` |
| FR-4 | Aggregate configured health signals into a periodic digest | Live — `myhealth-pulse` |
| FR-5 | Detect and tier longitudinal health changes by clinical significance | Live — `memory-diff` |
| FR-6 | Draft HIPAA §164.526 record-amendment requests and track response deadlines | Live — `request-amendment` |
| FR-7 | Ingest forwarded health email under a sender-allowlisted, transport-gated TripIt model | In progress — email router |
| FR-8 | Route each task to the most appropriate model for the deployment context | Partial — see [`../model-routing.md`](../model-routing.md) |
| FR-9 | Interact via Telegram (low-bandwidth); companion web UIs for richer review | Telegram live; `apps/` are Phase-1 fixtures |

## Non-functional requirements

- **PHI containment.** Health data stays in the user's local workspace; never
  in version control; never shared with third parties. Eval fixtures are
  synthetic.
- **Self-hostable on modest hardware.** Runs on a ~$30/mo single VM; supports a
  fully air-gapped, open-weight configuration.
- **Provider-agnostic.** No hard-coded model/provider; routes across Anthropic,
  OpenAI, Gemini, open weights, and Azure/NVIDIA/Palantir foundries.
- **Low-bandwidth and multilingual.** Telegram + email ingestion work on basic
  smartphones; skills treat language as a configurable parameter.
- **Eval-gated quality.** Every skill ships a Waza eval; the spec gate blocks
  merge (see [`../testing/eval-strategy.md`](../testing/eval-strategy.md)).
- **Auditable, draft-not-send safety.** Consequential clinical actions are
  human-in-the-loop by construction.
- **Cost-bounded.** Standard usage ~$35–50/mo; see [`../cost-guide.md`](../cost-guide.md).

## Out of scope (lives in Aria)

Per [`OPEN_CORE.md`](../../OPEN_CORE.md): the multi-tenant Patient Swarm
runtime, patient identity/SSO and RBAC, multi-tenant ingest routing, the
hospital-scale patient dashboard, the BAA-tier LLM gateway, per-tenant
append-only audit, hospital-scale EHR/FHIR integrations, and compliance
plumbing (BAA chain, SOC 2 evidence, HIPAA controls, FDA SaMD posture).

## Hard non-goals

Not a medical device, diagnostic system, treatment-recommendation engine,
emergency-response system, or a replacement for a clinician, EHR, or patient
portal. Out-of-scope items are named here so scope changes are deliberate.
