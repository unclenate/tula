# AGENTS.md

Cross-agent operating rules for this project. This file is read by Claude Code,
Cursor, Windsurf, GitHub Copilot, OpenAI Codex, and any other agent that
respects the AGENTS.md cross-client convention.

<!-- harness-managed-section -->

<!-- This section is maintained by .harness/platform/bootstrap/install.sh.
     Edits between the markers will be overwritten on re-bootstrap.
     See docs/adr/ADR-0001-submodule-integration.md for rationale. -->

## Harness governance

This repo adopts auto-harness for governance, mounted at `.harness/`.

- Active manifest: `harness.manifest.yaml`
- Governance rules: derived from active modules declared in the manifest
- Validators: `.harness/platform/validators/*.sh` (require Ruby 3.0+)
- Skills available: `.agents/skills/` (cross-client) and `.claude/skills/` (Claude Code)

Cross-agent operating rules come from the kernel trust model and active agent
packs declared in `harness.manifest.yaml`.

### Keeping the harness up to date

Periodically run `git submodule update --remote .harness` to pick up harness
improvements (new modules, validator fixes, new compositions). Review the
diff and commit. See `.harness/platform/workflow/maintenance-operations.md`
for the full upgrade workflow.

<!-- /harness-managed-section -->
