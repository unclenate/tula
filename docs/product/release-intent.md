# Release Intent

What Tula ships, how success is judged, and how it is distributed. Required by
`management/product-lite`.

**Owner:** @unclenate (RealActivity) | **Last Updated:** 2026-05-25

---

## What ships

Tula is distributed as an **open-source, self-hostable skill pack plus a
single-user reference deployment** — not a hosted service. A release comprises:

- The authored skills under `skills/` (six live: `health-records`, `med-pdf`,
  `epic-note`, `myhealth-pulse`, `memory-diff`, `request-amendment`), each with
  its Waza eval suite.
- The evaluation harness (`evals/`, `.waza.yaml`) and the continuously
  regenerated status doc `docs/evals.md`.
- Authoring conventions and deploy tooling (`skills/AGENTS.md`,
  `scripts/deploy-skills.sh`, VM install/preflight scripts).
- Deployment, security, cost, and routing documentation.
- The Phase-1 companion web apps (`apps/agent-studio`, `apps/my-aria`) on
  fixture data.

## Success criteria

- A self-hoster can stand up a working agent on a single VM in ~60–90 minutes
  following [`../deployment-guide.md`](../deployment-guide.md).
- Each live skill passes its Waza spec gate (`9/9`) and its `basic-usage` +
  `should-not-trigger` evals before release.
- Health data never leaves the user's environment in the reference deployment.
- A printed/emailed health document becomes correct structured FHIR R4 (the
  universal-photo-capture path) end to end.
- Recurring cost stays within the documented envelope (~$35–50/mo standard).

## Distribution and versioning

- **License:** Apache-2.0. Users bring their own API keys; no subscription, no
  lock-in. "Tula"/"Aria"/"RealActivity" are trademarks (see
  [`TRADEMARK.md`](../../TRADEMARK.md)).
- **Versioning:** the skill pack is versioned; operators pull a tagged release,
  review changes (esp. skill-behavior or security-posture changes), redeploy,
  and re-run the eval gate before relying on it. Breaking changes are called
  out in release notes. Operator upgrade flow:
  [`../deployment/self-hosting-guide.md`](../deployment/self-hosting-guide.md).
- **Downstream consumer:** Aria consumes Tula skills as a **versioned
  dependency**; improvements to public skills flow into Aria when it bumps its
  pin. Nothing proprietary flows back (see [`OPEN_CORE.md`](../../OPEN_CORE.md)).

## Out of scope for release

Hosted/multi-tenant operation, patient identity/SSO, hospital-scale EHR
integration, and compliance plumbing are Aria's release surface, not Tula's.
Contributions to those areas are not accepted in this repo.
