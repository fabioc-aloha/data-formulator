# Session Handoff — data-formulator (fork)

**Last updated**: 2026-07-22

**Full rules and rationale**: `fabio/RULES.md` (local-only per `.git/info/exclude`)

---

## TL;DR — where things stand

- **Production `ca-dataformulator`** in `rg-data-formulator` (GCX Survey Operations DEV) is **healthy on the previous known-good image** after a rolled-back MCP deploy attempt today.
- The MCP work is cleanly separated into three branches on `origin/` (this fork); it's **safe as an upstream PR** but **not safe to deploy as-is** without also carrying the fork's runtime/hardening commits.
- A **sister repo `fabioc-aloha/mcp-data-broker`** was created today for the enterprise-compliant MCP broker (Azure SQL / Fabric / semantic models). Only initial scaffold committed there.
- The **user-BYOI + broker architecture** is documented in `fabio/MCP-ARCHITECTURE.md` and `fabio/MCP-SISTER-REPO.md`.

## Branch topology (fork)

| Branch | Tip | Purpose |
|---|---|---|
| `main` | `fc297e7` | Historical fork mainline — do NOT reset until MCP work is proven end-to-end |
| `backup/pre-reconcile-2026-07-22` | `fc297e7` | Safety net (local + remote). Do not delete before 2026-08-22 |
| `contrib/governed-mcp-gateway` | `6f1917e` | Clean upstream PR source — `upstream/dev` + 1 squashed MCP commit. **Never** add `.github/**`, `.vscode/**`, `fabio/**`, or non-MCP work here |
| `reconcile/main-with-mcp` | `09b9918` (or newer if this handoff commit adds one) | Local working env — MCP + Alex brain + `.vscode` + `HANDOFF.md`. **Not safe to deploy** without additional cherry-picks from `backup` |
| `upstream/main`, `upstream/dev` | — | Microsoft read-only reference |

## Production state (`ca-dataformulator`)

| Item | Value |
|---|---|
| Public URL | https://data.gcxteam.com |
| Active revision | `ca-dataformulator--0000002` |
| Active image | `acrdataformulator.azurecr.io/data-formulator/web-dataformulator:azd-deploy-1784045589` (previous prod, rolled back to) |
| Dormant revision | `ca-dataformulator--0000001` — image `mcp-20260722-1804` (the MCP-integrated build, at 0% traffic, kept for reference) |
| Original revision | `ca-dataformulator--azd-1784046335` — was active before today's attempt |
| Replicas | 1 min / 1 max (unchanged) |
| Custom domain | `data.gcxteam.com` with managed TLS (unchanged) |

**AOAI (`aoai-dataformulator`)**: restored to locked-down state after local testing — `publicNetworkAccess=Disabled`, `disableLocalAuth=true`, no `ipRules`. **RBAC role assignment** for `fabioc@microsoft.com` (`Cognitive Services OpenAI User`) is still in place at the AOAI scope; harmless, leave it.

## PR #376 status

**Closed** on `microsoft/data-formulator` with a close comment pointing at
`fabioc-aloha:contrib/governed-mcp-gateway` as the replacement preview.
No new PR opened yet — user wanted to test MCP integration first before
resubmitting.

## Local dev state

- `.env` is minimized to Azure-only (AZURE_ENABLED=true, empty key = Entra path). Other providers disabled.
- **Local LLM calls against `aoai-dataformulator` fundamentally cannot work** from a laptop — `CloudGov_DisableLAOpenAI` policy blocks keys, and tenant policy blocks user tokens. Only managed identity from inside the VNet works. Confirmed in this session; do not re-litigate.
- No Data Formulator process running.
- `.venv` is up to date with reconcile branch deps (has `mcp[cli]>=1.2.0` from the reconcile commit).

## Sister repo `mcp-data-broker`

- Location: `../mcp-data-broker/` (sibling clone) and https://github.com/fabioc-aloha/mcp-data-broker (private)
- State: **initial scaffold only** — 20 files, 6 smoke tests passing, all real functionality stubs `NotImplementedError`
- Pilot target: **Azure SQL `CPE_Synapse` on `cpestaging.database.windows.net`**
- Full step-by-step next actions live in `../mcp-data-broker/HANDOFF.md`
- Design decisions locked: MIT license, bearer JWT caller auth, reuse `acrdataformulator`, dedicated Log Analytics
- Nothing provisioned in Azure yet for this repo (no RG, no Container App, no MSI)

## What to read first next session

Order matters — top-down:

1. **This file** (`HANDOFF.md`) — current state
2. `fabio/RULES.md` — branch strategy, deploy rules, commit hygiene, decisions log
3. `fabio/MCP-ARCHITECTURE.md` — user-BYOI end-to-end design
4. `fabio/MCP-SISTER-REPO.md` — sister repo scope + locked design decisions
5. `../mcp-data-broker/HANDOFF.md` — sister repo's own next-step order
6. `../mcp-data-broker/README.md` — sister repo product summary

Also loaded automatically by Copilot: `.github/copilot-instructions.md` (Alex Edition identity) and `.github/copilot-instructions.local.md` (points at `fabio/RULES.md`).

## Immediate next decisions (any of these unblocks progress)

| # | Decision | Effort | Blocking |
|---|---|---|---|
| **N1** | **Which piece of `mcp-data-broker` to implement first** — JWT validator, Azure SQL `list_tables`, or FastMCP wiring? See `mcp-data-broker/HANDOFF.md` § What's next | Small-medium | Progress on the pilot |
| **N2** | **Ask SQL AAD admin** to add `id-mcp-data-broker` MSI to `CPE_Synapse` (once the MSI exists) with `db_datareader`. See `mcp-data-broker/infra/README.md` § Manual prep | External wait | Any real end-to-end test |
| **N3** | **Register Entra app** for the broker (needed for `MCP_BROKER_AUDIENCE` — the JWT `aud` claim callers will use). Depends on tenant admin | External wait | JWT validation implementation |
| **N4** | **Decide upstream PR timing** — reopen `contrib/governed-mcp-gateway` → `microsoft:dev` now, or wait until the sister repo has a working pilot? | 0 (decision only) | Upstream visibility |
| **N5** | **Address `reconcile/main-with-mcp` deploy risk** — either declare it PR-only forever, or add the fork's runtime hardening commits so it's safe to deploy independently | Medium | Any future dev-env deploy of the reconcile branch |

## Do NOT do these without an explicit decision

- Re-deploy revision `--0000001` (the MCP-integrated image) — same regression risk exists until N5 is resolved
- Delete `backup/pre-reconcile-2026-07-22` — safety net until 2026-08-22
- Reset `origin/main` to `reconcile/main-with-mcp` — same reasoning; unsafe until N5
- Open the upstream PR before Chenglong is expecting it — coordination courtesy
- Modify `contrib/governed-mcp-gateway` in any way that isn't MCP-only code

## Key learnings burned in this session

1. **AOAI tenant policy `CloudGov_DisableLAOpenAI`** blocks `disableLocalAuth=false` across the whole tenant. API-key auth cannot be enabled on any AOAI in this tenant.
2. **User Entra tokens are also blocked at AOAI data plane** even with correct `Cognitive Services OpenAI User` role. Only managed identity tokens work. Root cause probably a tenant Conditional Access policy that we can't fix from here.
3. **"Azure SQL Entra delegated auth"** in the fork was an *attempted* feature blocked on unobtainable admin consent, not a working production feature. Env vars on the container reference it but nothing works. Safe to drop.
4. **Do not deploy `reconcile/main-with-mcp` without adding the runtime hardening commits** — I did this today and had to roll back. See RULES.md §11 (Decision log 2026-07-22).
5. **Multi-line commit messages must always go via `git commit -F <tempfile>`** — inline `-m` on PowerShell is unsafe (Backtick Hazard). Confirmed correct behavior on today's `chore: initial scaffold` commit in `mcp-data-broker`.
