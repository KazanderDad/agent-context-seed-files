---
description: First-run checklist for agents
alwaysApply: false
---

# agent-setup.md  

## Purpose 
Initialise the repository with required folders, hooks, and scaffolds.  
Agents execute these steps **if** they detect that `/agent-context/` is missing in the current repo, or if artefacts within it are missing.

---
## 1 Create agent-context artefacts (if absent)

### Seed mandatory files
  * Copy all files from `https://github.com/KazanderDad/agent-context-seed-files/tree/main/agent-context-mandatory`
  * Paste them into your local folder `/agent-context/`
  * E.g. use `bash npx -y degit "KazanderDad/agent-context-seed-files/agent-context-mandatory#main" agent-context`
  * FYI, these are the mandatory context files you should expect to copy, at minimum:
    * `session-log.md`      // The first agent run MUST set session-log.md → session: v0
    * `technical-spec.md`   // The first agent run should also populate the tech-spec with any knowledge it has about the app specs
    * `workflow.md`         // Comes prepopulated with content, only modify if requested to do so
  * Create `.env.example` (if missing) → one per line: `FOO-BAR=<value-placeholder>`
    * Never commit real secrets; the CI fails if .env is tracked.  
  * Create any other mandatory artefact referenced in **AGENTS.md**

### Seed optional files
  * Copy any relevant optional files from `https://github.com/KazanderDad/agent-context-seed-files/tree/main/agent-context-optional`
  * Paste them into your local folder `/agent-context/`
  * E.g. use `bash npx -y degit "KazanderDad/agent-context-seed-files/agent-context-optional#main" agent-context`

FYI, these are the optional context files you can expect to find in that folder:
| File                    | Use If the design of the app is systematic in the sense that you are asked to...             |
| ----------------------- | -------------------------------------------------------------------------------------------- |
| app-context.md          | ...document the background, business or other context prior to coding |
| functional-spec.md      | ...create a functional spec prior to coding |
| user-stories.md         | ...document the user stories prior to coding |
| dev-roadmap.md          | ...plan a roadmap prior to coding and only execute part of it |

---

## 2 Add Husky pre-commit hook

```bash
pnpm dlx husky-init && pnpm install
```

Replace generated hook with:

```bash
#!/usr/bin/env sh
. "$(dirname "$0")/_/husky.sh"
set -e  # stop on first failure

pnpm env:sync

# run JS + Solidity lint in parallel for speed
pnpm lint || echo "⚠️  Lint warnings"
pnpm spec:lint || echo "⚠️  Spec lint warnings"
```
Add to package.json:

```jsonc
"prepare": "husky install"
```

So clones/install automatically restore hooks

*Hook exits 0 unless env:sync fails (repo integrity).*

---

## 3 Add helper scripts

Add into scripts/
| File                    | Purpose                                                                                      |
| ----------------------- | -------------------------------------------------------------------------------------------- |
| **`env-sync.tsx`**      | Scan for `process.env.X` → append missing keys to `.env.example` (supports `${VAR}` syntax).  |
| **`spec-lint.tsx`**     | Ensure edited spec files bump their version to match the new session.                        |
| **`forge-coverage.sh`** | Wrapper for Foundry coverage (outputs to `./coverage/`).      //Optional                     |

Make the scripts executable: e.g. `chmod +x scripts/forge-coverage.sh`

Add scripts to `package.json`:
```jsonc
"scripts": {
  "dev": "next dev",
  "build": "next build",
  "start": "next start",

  "lint:js": "eslint . --ext .ts,.tsx",
  "lint:sol": "solhint 'contracts/**/*.sol'",
  "lint": "next lint && pnpm lint:sol",          // sol optional

  "env:sync": "tsx scripts/env-sync.ts",
  "spec:lint": "tsx scripts/spec-lint.ts",

  "test": "forge test",                          // optional
  "test:ui": "vitest run",                       // optional

  "coverage": "bash scripts/forge-coverage.sh",  // optional
  "db:diff": "supabase db diff --linked"         // optional

  /* prepare */ ,
  "prepare": "husky install"
}
```
*Note: Foundry / Forge / Coverage / lint:sol are optional and should only be used if the code uses Solidity.*

####  Seed example Foundry test

Create `contracts/evm/test/Example.t.sol`:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.30;

import "forge-std/Test.sol";

contract ExampleTest is Test {     // Example seed test. Replace with your actual contract test cases.
    function testTrue() public {
        assertTrue(true);
    }
}
```
*(Foundry auto‑detects any `*.t.sol` files under `test/`.)*

---

## 4 GitHub scaffolds
Copy the following files, if missing:

### CI workflow
  - Runs lint, env-sync, spec-lint, unit tests, schema drift check, secret scanning
  - Source file: `https://github.com/SmarTrust/smartrust_web/blob/main/.github/workflows/ci.yml`
  - Target file:  `.github/workflows/ci.yml`

### Supabase nightly workflow  (mandatory if using supabase)`
  - Runs remote supabase schema drift check
  - Source file: [supabase-nightly.yml](https://raw.githubusercontent.com/KazanderDad/agent-context-seed-files/refs/heads/main/.github/workflows/supabase-nightly.yml)
  - Target file:  `.github/workflows/supabase-nightly.yml

### PR template
  - Checklist defined in the Appendix below.
  - Source file: [PULL_REQUEST_TEMPLATE.md](https://raw.githubusercontent.com/KazanderDad/agent-context-seed-files/refs/heads/main/.github/PULL_REQUEST_TEMPLATE.md)
  - Target file: `.github/PULL_REQUEST_TEMPLATE.md`

### Design Freeze (optional)
  - Action to block PRs labelled `design-freeze`.
  - Source file: [design-freeze.yml](https://raw.githubusercontent.com/KazanderDad/agent-context-seed-files/refs/heads/main/.github/workflows/design-freeze.yml)
  - Target file: `.github/workflows/design-freeze.yml`

---

## 5 Commit & log

* Agent commits with message `chore(setup): bootstrap agent tooling`
* Bootstrap complete; switch to regular session using `workflow.md`.

---
