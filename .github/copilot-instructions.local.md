# Identity (heir-owned)

<!-- This file is heir-owned. Edition upgrades never overwrite it. -->

## Project Context

Data Formulator is an interactive AI-powered data analysis system by Microsoft
Research. Users connect to data (files, warehouses, cloud connectors), then use
natural language to explore and visualize it. Backend is Python 3.11+ (Flask),
frontend is React 18 + TypeScript + Vite + MUI + Redux.

### Python Environment

- Always use `uv` instead of `pip` for installing packages (e.g. `uv pip install`, `uv pip install -e .`).
- Use `uv run` to execute Python scripts and modules (e.g. `uv run python script.py`, `uv run pytest`).
- The virtual environment is at `.venv/`. Activate it with `source .venv/bin/activate` if needed.
- The Python source is under `py-src/`. The project is installed in editable mode via `uv pip install -e .`.

### Node / Frontend

- Yarn v1.22.22 only per `df-package-manager-conventions.instructions.md`. Never
  `npm install` or `pnpm`.

### Repository Layout

- `py-src/data_formulator/` — Flask backend (routes, agents, data loaders, MCP gateway)
- `src/` — React + TypeScript frontend
- `tests/backend/` — pytest suite (see `df-backend-test-conventions.instructions.md`)
- `tests/frontend/` — Vitest suite (see `df-frontend-test-conventions.instructions.md`)
- `docs/dev-guides/` — canonical technical guidance; read the matching guide
  before implementing per `df-dev-guides-first.instructions.md`

## Local Working Rules

The complete ruleset for how this fork is organized — branch topology, upstream
contribution rules, include/exclude paths, personal content boundaries, commit
hygiene, sync workflow, testing rules, and the decisions log — lives in
`fabio/RULES.md`.

**Read `fabio/RULES.md` at the start of any session that touches**:

- Branch strategy (main / backup / contrib / reconcile)
- Upstream PR authoring or the closed PR #376
- Anything under `py-src/data_formulator/mcp_gateway/` or `mcp/`
- The Alex ACT Edition brain restore (2026-07-22 commit `d161683`)
- `fabio/` folder contents

### `fabio/` availability caveat

`fabio/` is repo-local ignored via `.git/info/exclude` and **does not exist on
fresh clones**. If a session starts and `fabio/RULES.md` is missing:

1. Ask the user whether they want to restore it from another machine, or
2. Bootstrap a new copy with:

   ```pwsh
   Add-Content -Path .git/info/exclude -Value "`nfabio/`n"
   New-Item -ItemType Directory -Path fabio -Force | Out-Null
   ```

   Then re-derive the rules interactively (do NOT auto-generate; the rules
   contain user decisions, not defaults).

### Quick reference (safe fallback if `fabio/RULES.md` is absent)

- **Upstream target**: PRs go to `microsoft/data-formulator:dev` (not main). Per
  maintainer Chenglong's preference on the closed PR #376.
- **Upstream scope**: only the governed MCP gateway is upstream-worthy. Never
  bundle personal tooling, environment-specific hardening, or Azure deployment
  infra into an upstream PR.
- **Branch roles**:
  - `contrib/governed-mcp-gateway` — clean upstream PR source. Never add
    `.github/**`, `.vscode/**`, `fabio/**`, or non-MCP work here.
  - `reconcile/main-with-mcp` — local working env (MCP + Alex brain +
    `.vscode`). Will become `main` after MCP integration testing is complete.
  - `backup/pre-reconcile-2026-07-22` — safety net for the pre-reconcile fork
    main state. Do not delete before 2026-08-22.
- **Never merge `upstream/main` directly into fork `main`.** Reconciliation is
  always via topic-branch cherry-pick to avoid the 49-file conflict surface.
- **Multi-line commit messages** — write to `$env:TEMP\<slug>.txt`, then
  `git commit -F <file>`. Never inline for anything with backticks, dollar
  signs, em-dashes, or newlines (PowerShell Backtick Hazard).
- **Destructive operations need explicit approval** — hard-resetting `main`,
  force-pushing, deleting `backup/*`, or opening/closing upstream PRs. See
  section 8 of `fabio/RULES.md`.
- **Session continuity** — cross-session handoffs go to repo-root `HANDOFF.md`,
  never to `/memories/session/` (that tier clears at end of conversation).

## My Preferences

<!-- Communication style, naming conventions, test framework choices, etc. -->
