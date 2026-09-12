# Adapter: OpenAI Codex

Install with `./scripts/init codex`.

Codex reads the repository's root `AGENTS.md` natively. This adapter therefore does not copy the
rules into another instruction file: `AGENTS.md` remains ANEW's single source of truth. The
adapter adds only Codex-specific wiring:

- `.agents/skills/anew-workflow/` makes the ANEW workflow available as `$anew-workflow` (or
  from `/skills`) for repeatable feature, bug, review, and verify tasks.

- `.codex/config.toml` keeps the project in an approval-gated, workspace-write sandbox and raises
  the project instruction size limit so the complete ANEW signpost is available.
- `.codex/agents/reviewer.toml` defines a read-only reviewer for the independent REVIEW step.
- `.codex/rules/anew.rules` blocks force pushes, hard resets, rebases, and recursive removal when
  Codex requests to run those commands outside the sandbox.

Project-local `.codex/` configuration is loaded by Codex only after the project is trusted. The
rules file is an additional command guard; it does not replace `AGENTS.md`, `scripts/check`, CI,
or a human approval gate. Codex's command rules govern commands outside the sandbox, so keep the
workspace sandbox enabled and do not treat the rules as a substitute for version control or review.

After installation:

```bash
codex doctor
codex "Read AGENTS.md and summarize the active ANEW gates."
```

In an interactive Codex session, invoke `$anew-workflow` (or open `/skills`) before describing
the work. This loads the matching ANEW workflow without duplicating its rules.

For an independent review, ask Codex to use the project `reviewer` agent with only the diff, the
spec, and the relevant project files as context. The reviewer runs in a read-only sandbox and must
not edit, commit, or delete files.
