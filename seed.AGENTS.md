---
description: Authoritative guide for all software-writing agents in this repository
alwaysApply: true
---

# AGENTS.md  
---

## 0 Philosophy

| Principle | Meaning for agents |
|-----------|-------------------|
| **Data before UI** | Treat database schema, migrations, and existing rows as sacred: can be updated, but carefully and structured. |
| **Small, safe steps** | Every session past v1 is a micro-iteration with its own log, spec delta, and reflection. |
| **Design, then develop** | Every session both amends the design specs and implements the changes into code. |
| **Canadian English** | House style for all prose and code. |
| **Security first** | No plaintext secrets; use environment variables referenced by pattern in `.env.example`. |
| **Test a lot** | Configure solidity-coverage and integration tests to catch bugs early. Use CI workflows to run tests on every pull request. Foundry’s forge coverage is wired via scripts/forge-coverage.sh; keep ≥ 90 % line coverage. |

---

## 1 Repository Structure
Source files are organized under the `app/` directory while context files for agents are organized under agent-context/ with the following layout:
```
.github/workflows/       # Mandatory - PR template + CI workflows for build, lint, supabase, secret scanning
agent-context/           # designs, logs and other context files for developers and agents
├─ session-log.md        # Mandatory – append-only per session, new entries at top of file, v ↑ per edit
├─ technical-spec.md     # Mandatory – latest technical spec
├─ functional-spec.md    # Mandatory – latest functional spec
├─ user-stories.md       # Optional – personas & acceptance criteria
├─ app-context.md        # Optional – problem, approach, value prop
├─ sql-diff-v{N}.sql     # Optional – Auto-generated incremental SQL patches - append-only per session (new entries at top of file)
└─ workflow.md           # Repetitive per-session checklist
app/                     # app router
components/              # shared components
hooks/                   # custom React hooks
lib/
public/                  # static assets to be served
scripts/                 # helper scripts for env:sync, lint, etc
styles/                  # global styles
supabase/                # latest sql schema (github action runs db pull on PR)
test/
README.md                # intro to the project and this repo
AGENTS.md                # this file
agent-setup.md           # One-time boot-strap guide for agents
```

*Note: If expected files are missing from .github/workflows or from agent-context/ then follow [these instructions](https://raw.githubusercontent.com/KazanderDad/agent-context-seed-files/refs/heads/main/agent-setup.md) to create them.*

---

## 2 Coding Conventions

* **Framework & Versions**
  * Nextjs v15.3
  * Nodejs v24
  * Foundry (forge + cast) & Hardhat for coverage; Solidity ^0.8.30
* **Structure**
  * Add or modify files within the structure above if possible.
  * If you need to add folders then also update both AGENTS.md (this file) and README.md folder diagrams.
* **Lint / Format**  
  * **TypeScript** → ESLint + Prettier  
  * **Solidity**   → `solhint` (or `solidity-lint`) + Prettier plugin  
  * `pnpm lint` runs **both** (`pnpm lint:ts && pnpm lint:sol`)
* **Tabs / Indent**
  * Four spaces in solidity, two spaces in typescript (no hard tabs)  
* **Env handling**
  * Update `.env.example` with *names* of new vars (never values).
  * Boolean flags must be 'true'/'false' strings to avoid docker‑compose parsing quirks. 
* **Secrets scan**
  * Detect hard-coded keys and secrets → Refactor to env vars + update `.env.example` + note in session log need for adding new env var
  * Add logs where needed, but ensure no sensitive values (tokens, IDs, secrets) are logged even in dev mode.  
  * Run trufflehog on diff in CI workflow and full scan in nightly workflow
* **Testing**
  * If any new logic →  Always add or modify unit tests accordingly.  
  * Jest / Vitest for JavaScript units
  * Foundry for Solidity (if present)
  * Cypress for front-end e2e (if present).
* **Pull Requests**
  * Keep pull request descriptions short, following [conventionalcommits](https://www.conventionalcommits.org/en/v1.0.0/)
  * If any related issues are known, mention them in PR (e.g. "Isses: #10, #11")
* **Session version tags**
  * Follow SemVer (v2.0.0-alpha)
  * Major IDs ("v1.0", "v2.0") mirror new feature branches
  * Minor IDs ("v2.0", "v2.1") mirror different coding sesssions on the same feature branch

---

## 3 Process Overview

| Phase | File | Detail |
|-------|------|--------|
| **Bootstrap (first run)** | `https://raw.githubusercontent.com/KazanderDad/agent-context-seed-files/refs/heads/main/agent-setup.md` | Creates folders, Husky hooks, CI scaffold, etc. Installs git-moji-cli for commit emojis (optional). |
| **Every session** | `workflow.md` | Mandatory checklist (log, spec update(s), code, summary). |
| **Artefact maintenance** | Scripts inside `scripts/` | `env-sync.ts`, `spec-lint.ts`, etc. |
| **CI Triggers** | Pushes to main and all PRs run forge test, forge coverage, pnpm lint, and hardhat size-contracts. |

Agents **must** read [agent-setup.md](https://raw.githubusercontent.com/KazanderDad/agent-context-seed-files/refs/heads/main/agent-setup.md) if artefacts are missing, otherwise follow `workflow.md` each time.

---

## 4 Guard-rails

* SQL migrations must be **idempotent & reversible** (include -- DOWN section which must revert exactly to previous schema; each DROP/ALTER should be preceded by IF EXISTS/IF NOT EXISTS..).
* Pre-commit hook **warns** (not blocks) if ESLint and similar (e.g. in Foundry) cannot start.
* Session log must bump version +1 for each session
* All new external calls must use SafeERC20 / Address.functionCall and be covered in tests.
