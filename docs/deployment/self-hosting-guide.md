# Self-Hosting Guide

The operator's contract for running Tula. Tula ships as published Apache-2.0
software the **user operates** on their own infrastructure — there is no hosted
service. Required by `delivery/self-hosted-oss`.

**Owner:** @unclenate (RealActivity) | **Last Updated:** 2026-05-25

---

## Who operates this

**You do.** Whoever deploys Tula is its operator and is responsible for its
security, data, and upkeep. RealActivity does not run a hosted instance; each
deployment is independent and self-contained. (The multi-tenant, operator-run
edition is Aria — a separate product; see [`OPEN_CORE.md`](../../OPEN_CORE.md).)

## Minimum viable deployment

- **Host:** Azure B2s VM (2 vCPU, 4 GB RAM), Ubuntu Server 24.04 LTS. ~$30/mo.
  Other clouds or bare metal work; the VM is an orchestration host, not a
  compute node.
- **Runtime:** [OpenClaw](https://github.com/openclaw/openclaw), reachable via
  Telegram.
- **Model access:** at least one provider API key (Anthropic by default; Gemini
  for web search; optional OpenAI/local). Open-weight local models (MedGemma 4B
  via vLLM) enable an air-gapped, no-API-fee deployment.
- **Skills:** deploy with `scripts/deploy-skills.sh` to
  `~/.openclaw/workspace/skills/`. Provisioning helpers:
  `scripts/install-health-skillz-vm.sh`, `scripts/health-skillz-vm-preflight.sh`,
  `scripts/set-openclaw-health-skillz-env.sh`.

Full step-by-step (including SSH key setup and first-boot fixes) is in
[`deployment-guide.md`](../deployment-guide.md). VM hosting specifics are in
[`health-skillz-vm-hosting.md`](../health-skillz-vm-hosting.md).

## Data locations the operator owns

All under `~/.openclaw/workspace/` on the VM, **local only**, never in version
control:

- `skills/` — the deployed skill pack
- `memory/` — dated free-text daily notes the agent writes
- `.health-records-cache/` — FHIR R4 resources (per-day, per-provider)
- `.med-pdf-cache/`, `.myhealth-pulse-cache/`, `.memory-diff-cache/` — per-skill
  working data
- Provider API keys and the OpenClaw config — operator-managed secrets

## Security posture the operator inherits

Tula secures the ingestion and exfiltration paths **by default**; the operator
must secure the host and the credentials. See
[`security-model.md`](../security-model.md) for the full threat model.

**Shipped by the project:**
- Inbound: Exchange Online transport allowlist — only authorized senders reach
  the mailbox (SMTP-layer, before any processing). SPF/DKIM/DMARC anti-spoofing.
- Outbound: recipient allowlist at the transport layer — the data-exfiltration
  control even if an instruction is injected.
- Application-level sender re-check in the email router (defense in depth).
- Local-only storage; draft-never-send for clinical communications.

**The operator must secure:**
- The VM (SSH keys, OS patching, firewall — only required ports open).
- Provider API keys and OpenClaw config (treat as secrets; never commit).
- DKIM/DMARC configuration on the authorizing domain (set DMARC to `reject` or
  `quarantine`).

Residual risk is documented honestly in `security-model.md` (notably indirect
prompt injection via forwarded third-party content). If this deployment handles
data that matters — it handles PHI — create `docs/security/risk-register.md`
(criticality ≥ medium expects it).

## Upgrade and versioning

Tula is a versioned, distributable skill pack. Pull a tagged release, review
the changes (especially any skill behavior or security-posture changes), redeploy
skills with `scripts/deploy-skills.sh`, and re-run the eval gate
(`waza check`) before relying on the update. Breaking changes are called out in
release notes. See [`docs/product/release-intent.md`](../product/release-intent.md).

## Backup and recovery

`scripts/agent-backup.sh` snapshots the agent workspace and commits/pushes it
with a regex secret-scan gate (it refuses to commit detected secrets). Restore
by re-provisioning the VM, redeploying skills, and restoring the workspace
snapshot. Back up: the workspace caches, `memory/`, and the OpenClaw config;
**not** raw API keys (re-issue those).

## Costs

Standard usage ~$35–50/mo (VM + Claude/Gemini API); intensive (imaging,
genomics, voice) ~$51–115/mo. A fully self-hosted MedGemma + local-LLM
configuration can reduce recurring cost to the VM alone (~$30/mo). Full
breakdown and routing-based scenarios in [`cost-guide.md`](../cost-guide.md).
Set spending limits on provider dashboards.
