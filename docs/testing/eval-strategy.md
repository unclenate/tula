# Eval Strategy

Tula's primary quality gate is **binary-graded evaluation of skill behavior**,
run by [Microsoft Waza](https://github.com/microsoft/waza), not line/branch
coverage. Required by `management/eval-gated-testing`.

**Owner:** @unclenate (RealActivity) | **Last Updated:** 2026-05-25 | **Runner:** Microsoft Waza

---

## What we evaluate

The units under evaluation are the **authored skills** (`skills/<name>/`). Each
skill's behavior — does it do the right thing, in the right format, and decline
when it should — is what gates merge. Deterministic helper scripts (`.mjs`) are
exercised through the skills that call them. Quality here is behavioral
correctness of an agent skill, which line coverage cannot express; hence evals,
not coverage.

## Runner and CI gate

- **Runner:** Waza, configured in [`.waza.yaml`](../../.waza.yaml) (executor
  `copilot-sdk`, default model `claude-sonnet-4.6`, per-task timeout 300s,
  token warning threshold 500).
- **Local gate:** `waza check skills/<skill-name>` before deploy (see
  [`skills/AGENTS.md`](../../skills/AGENTS.md) § Validation). Acceptable
  outcome: `Spec Compliance: 9/9 passes`.
- **CI gate:** [`.github/workflows/eval-status.yml`](../../.github/workflows/eval-status.yml)
  runs on every PR touching `skills/**`, `evals/**`, or the eval-status script,
  and regenerates the public status doc `docs/evals.md`. A skill that does not
  pass the spec gate does not ship.

## Eval layout

Each skill has an `evals/<skill>/` directory mirroring `skills/`:

- `eval.yaml` — metric, threshold, and graders for the skill
- `tasks/` — one YAML per task (id, name, tags, inputs, expected output)
- `fixtures/` — synthetic test data

## Graders and thresholds

Graders assert task adherence, not mere liveness:

| Grader | Asserts |
|--------|---------|
| `task_completion` | The skill accomplished the task; gated at **≥ 0.8** |
| `not_empty` | Output is present (floor check, never the only grader) |
| `under_word_budget` | Drafts stay within budget (e.g. portal messages < 400 words) |

Thresholds are quality commitments. Lowering a threshold or removing an
anti-trigger case requires a change-log entry, an ADR, or human approval —
agents must not weaken the gate unilaterally (`management/eval-gated-testing`
companion rule).

## Task taxonomy

Every skill carries at least a `basic-usage` and a `should-not-trigger` case.
The taxonomy encodes *intent* coverage:

| Class | Purpose |
|-------|---------|
| `basic-usage` | Expected behavior on the common input |
| `edge-case` | Boundary / partial / ambiguous input |
| `should-not-trigger` | Anti-trigger: the skill must decline or stay silent |
| `triage-override` (clinical) | A red-flag/safety gate that must fire and pre-empt normal output |

The `should-not-trigger` and `triage-override` classes are what distinguish an
eval gate from a demo: they assert the agent knows when **not** to act —
critical for a patient-facing health agent.

## Fixtures: synthetic only

Eval fixtures use a synthetic persona (e.g. `@drsynth` / "Dr. Casey Synth") —
no real names, handles, providers, PHI, or production records under version
control. `skills/request-amendment/scripts/redact_phi_for_eval.mjs` enforces
de-identification for any fixture derived from real material. This is the
testing-side expression of operating-principle § 2 (PHI never enters the repo).

## Flake policy

LLM-output evals are non-deterministic. Treat a single red run as advisory;
require the spec gate to pass on the merge-candidate commit. Model upgrades are
a known regression source (behavior changes with no code change) — re-run the
full suite when the default model in `.waza.yaml` changes, and review
`docs/evals.md` for drift.

## Relationship to coverage

This posture is a sibling to `management/testing-standard`, not a replacement.
Where deterministic, non-agent code accrues (e.g. the Next.js apps gaining
logic), conventional test coverage still applies; the eval gate governs the
agent-skill surface.
