---
name: anew-workflow
description: Use when working in an ANEW repository on a feature, bug fix, refactor, incident, bootstrap, review, or verification task. It connects the request to the repository's spec, plan, build, review, triage, verify, and ship workflow.
---

# ANEW workflow for Codex

Use this skill together with the repository root `AGENTS.md`. `AGENTS.md` is the source of truth;
do not replace, weaken, or restate its invariant rules here.

1. Identify the request type and open the matching file under `workflows/`.
2. For new work, create or update a spec under `specs/active/` before changing application code.
3. Keep the implementation plan under `specs/plans/` and stop at the human approval gate.
4. Build only after approval, keeping the diff inside the approved scope.
5. Review from the diff and spec in a fresh Codex session or read-only reviewer context.
6. Surface findings for human triage; do not silently dismiss or fix around a finding.
7. Run `./scripts/check` and map every acceptance criterion to evidence before delivery.
8. Never edit files under `specs/done/`; use the matching recovery ramp in `prompts/recovery/` if
   the spec or plan must change.

Useful entry points:

- `workflows/bootstrap.md` for a new or unconfigured workspace
- `workflows/feature-development.md` for a feature
- `workflows/bug-fix.md` for a bug
- `workflows/refactor.md` for a refactor
- `workflows/incident.md` for an incident
- `prompts/review.md` and `prompts/verify.md` for independent review and proof

At the end, report the commands run, their results, the changed files, the acceptance-criterion
evidence, and the remaining uncertainty. “Done” without that evidence is not a valid delivery.

