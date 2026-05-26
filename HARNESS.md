# HARNESS.md

This project uses the modular harness manifest at `harness.manifest.yaml`.
Auto-harness is mounted at `.harness/` as a git submodule.

- Governance: `.harness/platform/workflow/submodule-integration.md`
- Validators: `.harness/platform/validators/`
- Skills (symlinked): `.agents/skills/` and `.claude/skills/`

Update the submodule to pull in upstream improvements:

```
git submodule update --remote .harness
```
