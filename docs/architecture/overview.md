# Architecture Overview

How Tula is structured: where the agent runs, how skills are loaded, where data
lives, and the trust boundaries between them. Required by
`architectures/web-app` and `architectures/agent-skill-pack`.

**Owner:** @unclenate (RealActivity) | **Last Updated:** 2026-05-25

---

## Topology at a glance

```
Phone camera / forwarded email      Telegram / web
        |                                 |
   Exchange Online                   (notifications,
   transport allowlist                chat, capture)
        |                                 |
        v                                 v
+----------------------------------------------------+
|  Single self-hosted VM (Azure B2s, Ubuntu 24.04)   |
|                                                    |
|   OpenClaw runtime  ──loads──>  Tula skill pack    |
|        |                        (~/.openclaw/        |
|        |                         workspace/skills/) |
|        v                                            |
|   workspace memory + FHIR R4 caches (local only)   |
+----------------------------------------------------+
        |  (model-provider seam — deployment-context-aware)
        v
  Claude / OpenAI / Gemini / vLLM open-weights · Azure / NVIDIA / Palantir foundries
```

The reference deployment is one user, one VM. The heavy computation happens at
the model-provider level; the VM is an orchestration host. See
[`deployment-guide.md`](../deployment-guide.md) for the step-by-step build and
[`docs/deployment/self-hosting-guide.md`](../deployment/self-hosting-guide.md)
for the operator contract.

## Agent runtime location

The agent loop runs **on the user's VM**, inside the
[OpenClaw](https://github.com/openclaw/openclaw) runtime — not in the browser
and not in a hosted multi-tenant control plane. Prompt assembly, tool
selection, and skill execution all happen server-side on infrastructure the
user controls. (The multi-tenant control plane that schedules one agent per
patient is Aria, a separate product — see [`OPEN_CORE.md`](../../OPEN_CORE.md).)

## Skill-loading model (the unit of product)

Tula's product is an **authored skill pack**. Skills are developed in this repo
under `skills/<name>/` (a `SKILL.md` spec + `references/` + Node ESM `scripts/`),
authored to the conventions in [`skills/AGENTS.md`](../../skills/AGENTS.md), and
**deployed** to `~/.openclaw/workspace/skills/` on the VM via
`scripts/deploy-skills.sh`. OpenClaw loads each `SKILL.md` and invokes skills by
description match. The six live skills: `health-records` (patient-authorized
SMART-on-FHIR client), `med-pdf`, `epic-note`, `myhealth-pulse`, `memory-diff`,
`request-amendment`. Governance of the authored pack is
`architectures/agent-skill-pack`; the eval gate that protects it is
`management/eval-gated-testing`.

## Action surface and permission boundary

- **Workspace-scoped caches.** Each skill writes only to its own cache under
  the workspace (`.health-records-cache/`, `.med-pdf-cache/`, per-skill dated
  files). PHI is confined here and never enters version control.
- **Draft-never-send.** Clinical communications (portal messages via
  `epic-note`, amendment requests via `request-amendment`) are produced as
  drafts a human sends. The agent has no authority to transmit to a provider.
- **Triage-first.** `epic-note` scans for emergency red flags and can return a
  911 redirect before any draft is produced.
- **Ingestion boundary.** Inbound data arrives through the email router under a
  TripIt-style model: the user forwards documents to a dedicated mailbox locked
  to a sender allowlist at the Exchange transport layer, before any model sees
  the content. See [`security-model.md`](../security-model.md) and
  [`email-router-design.md`](../email-router-design.md).

## Data storage

All health data is stored **locally** on the user's VM as FHIR R4 resources in
the workspace caches, plus free-text daily memory notes. No cloud health
platform, no third-party data sharing. Structured outputs (FHIR R4, lab JSON,
imaging JSON, audit traces) are shaped to land cleanly into downstream
ontologies when an operator chooses to export.

## Web surface (browser/server boundary)

Two Next.js 15 (App Router, TypeScript) companion apps live under `apps/`:
`agent-studio` (activity-feed view of ingested health data) and `my-aria`
(MyChart-style patient portal). Both are **Phase-1 walking skeletons running on
fixture data** — they do not yet connect to a live workspace or hold
credentials. When they gain a live data path, the trust boundary is explicit:
the agent runtime and PHI stay server-side; the browser receives only rendered,
authorized views. Any future in-product agent chat surface would additionally
activate `domains/agentic-interfaces`.

## Model-provider seam

Skills do not hard-code a provider. Each task is routed to the most capable,
cost-effective, and privacy-appropriate model available in the deployment
(Anthropic / OpenAI / Gemini / open-weights via vLLM; Azure / NVIDIA / Palantir
foundries; healthcare models such as MedGemma / MedImageInsight / MedASR). The
routing matrix and fallback chains are in
[`model-routing.md`](../model-routing.md). Traces are OpenTelemetry-shaped for
export to any compatible collector.
