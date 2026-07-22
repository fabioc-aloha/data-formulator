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

## My Preferences

<!-- Communication style, naming conventions, test framework choices, etc. -->
