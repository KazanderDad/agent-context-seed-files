# agent-setup.md  
_First-run checklist for agents_

## Purpose 
Initialise the repository with required folders, hooks, and scaffolds.  
Agents execute these steps **only if** they detect that `agent-context/` or mandatory artefacts within it are missing.

---
## 1 Create base artefacts (if absent)

1. **Folder** `agent-context/`
2. **Seed files** (version `v1`)
   * `agent-context/session-log.md`

     ```markdown
     ---
     session: v1
     timestamp: <ISO 8601 America/Toronto>
     agent: OpenAI‑Codex
     ---
     _Bootstrap session started._
     ```

   * `agent-context/spec.md` → header as per **AGENTS.md**  
   * `.env.example` → one per line: `FOO=<value>` (never real secrets)  
   * Create **any** other mandatory artefact referenced in **AGENTS.md**

---

## 2 Add Husky pre-commit hook

```bash
npx husky-init && pnpm install
```

Replace generated hook with:

```bash
set -e
#!/usr/bin/env sh
. "$(dirname "$0")/_/husky.sh"

pnpm env:sync
pnpm spec:lint || echo "⚠️  Spec lint warnings"
CI=true pnpm lint     || echo "⚠️  ESLint issues – job allowed"
```

*Hook exits 0 unless env:sync fails (repo integrity).*

---

## 3 Add helper scripts

scripts/
| File                    | Purpose                                                                                      |
| ----------------------- | -------------------------------------------------------------------------------------------- |
| **`01-env-sync.ts`**    | Scan for `process.env.X` → append missing keys to `.env.example` (supports `${VAR}` syntax). |
| **`02-spec-lint.ts`**   | Ensure edited spec files bump their version to match the new session.                        |
| **`forge-coverage.sh`** | Wrapper for Foundry coverage (outputs to `./coverage/`).                                     |

  Make the script executable: `chmod +x scripts/forge-coverage.sh`

Add to `package.json`:
```jsonc
"scripts": {
  "lint:js": "eslint . --ext .ts,.tsx",
  "lint:sol": "solhint 'contracts/**/*.sol'",
  "lint": "pnpm lint:js && pnpm lint:sol",
  "env:sync": "ts-node scripts/env-sync.ts",
  "spec:lint": "ts-node scripts/spec-lint.ts",
  "test": "forge test",                // Foundry unit tests
  "coverage": "bash scripts/forge-coverage.sh"
}
```
####  Seed example Foundry test

Create `contracts/evm/test/Example.t.sol`:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.30;

import "forge-std/Test.sol";

contract ExampleTest is Test {
    function testTrue() public {
        assertTrue(true);
    }
}
```
*(Foundry auto‑detects any `*.t.sol` files under `test/`.)*

---

## 4 GitHub scaffolds
If .github/PULL_REQUEST_TEMPLATE.md or .github/workflows/ci.yml (or design-freeze.yml) are missing, create them using the stubs defined in the Appendix below.
1. **CI workflow** `.github/workflows/ci.yml`
   * Runs lint, env-sync, spec-lint, unit tests, schema drift check, secret scanning
2. **PR template** `.github/PULL_REQUEST_TEMPLATE.md`
   Checklist defined in the Appendix below.
3. **(Optional)** `design-freeze.yml` action to block PRs labelled `design-freeze`.

*(Full stubs in Appendix 1–3.)*

---

## 5 Commit & log

* Agent commits with message `chore(setup): bootstrap agent tooling`
* Append **Summary** & **Reflections** to `session-log.md` as defined in `workflow.md`.

*Bootstrap complete; switch to regular sessions using `workflow.md`.*

---

## Appendix 1: Scaffold PR Template

<!-- .github/PULL_REQUEST_TEMPLATE.md -->
```markdown
### ☑️  PR Checklist

- [ ] **Session log**:
      - [ ] Version bumped
      - [ ] Appended session summary at top of `agent-context/session-log.md` for **v{N}**
      - [ ] Reflections section completed, including one confusion and one suggestion to the next agent
- [ ] **Spec files**: Relevant spec(s) updated and bumped to version **v{N}**
- [ ] **Agent instructions**: AGENTS.md updated with relevant changes, if any
- [ ] **Lint**: `pnpm lint` passes *or* ESLint failure noted in commit
- [ ] **Tests**: New or affected logic covered by unit tests
- [ ] **Environment**: `.env.example` updated with any new variable names
- [ ] **SQL changes,** if any:  
      - [ ] Matching `sql-diff-v{N}` with **UP / DOWN** blocks committed  
      - [ ] I have (or will) apply the patch in Supabase _or_ open a follow‑up task

_If this PR is subject to a **design-freeze** label, remove the label before merging._
```

---

## Appendix 2: Scaffold Minimal CI workflow
<!-- .github/workflows/ci.yml -->
```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
  workflow_dispatch:        # you can manually trigger the whole pipeline

defaults:
  run:
    shell: bash

########################################################################
# 1.  Build / lint / tests
########################################################################
jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      # 1 – Check out code
      - uses: actions/checkout@v4

      # 2 – Install pnpm (and wire up its cache)
      - name: Setup pnpm
        uses: pnpm/action-setup@v2
        with:
          version: 10
          run_install: false

      # 3 – Install Node and enable dependency caching
      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: pnpm
          cache-dependency-path: '**/pnpm-lock.yaml'

      # 4 – Install project dependencies
      - name: Install deps
        run: pnpm install --frozen-lockfile

      # 5 – Lint, spec, and ESLint checks
      - name: Lint & Spec checks
        run: |
          pnpm env:sync
          pnpm spec:lint
          pnpm lint || echo "⚠️ ESLint failed to start — allowed to pass"

########################################################################
# 2.  Supabase schema refresh job (runs only on pushes to main or manual)
########################################################################
  schema:
    if: github.ref == 'refs/heads/main'   # skip on PR branches
    needs: build                          # run after build job
    runs-on: ubuntu-latest
    env:
      SUPABASE_PROJECT_REF: ${{ secrets.SUPABASE_PROJECT_REF }}
      SUPABASE_DB_PASSWORD: ${{ secrets.SUPABASE_DB_PASSWORD }}
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    concurrency: {group: schema-refresh, cancel-in-progress: true}

    steps:
      - uses: actions/checkout@v4
      - timeout-minutes: 5
      - run: |
          curl -sL https://github.com/supabase/cli/releases/latest/download/supabase_linux_amd64.tar.gz \
          | sudo tar -xz -C /usr/local/bin supabase
      - id: pull
        run: |
          DB_URL="postgres://postgres:${SUPABASE_DB_PASSWORD}@${SUPABASE_PROJECT_REF}.supabase.co:6543/postgres"
          supabase db pull --db-url "$DB_URL" --schema public > tmp_schema.sql
          if diff -q tmp_schema.sql supabase/schema.sql >/dev/null; then
            echo "changed=false" >> "$GITHUB_OUTPUT"
          else
            echo "changed=true"  >> "$GITHUB_OUTPUT"
          fi
      - if: steps.pull.outputs.changed == 'true'
        run: |
          mv tmp_schema.sql supabase/schema.sql
          git config user.name  github-actions
          git config user.email github-actions@users.noreply.github.com
          git add supabase/schema.sql
          git commit -m "chore(schema): refresh Supabase dump"
          git push


########################################################################
# 3.  TruffleHog Scanning For Exposed Secrets and Keys
########################################################################
  trufflehog:
    name: Secret scan
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        # For PRs from forks, fetch only the diff to keep token scope minimal
        with:
          fetch-depth: 0

      - name: Run TruffleHog on HEAD^..HEAD
        uses: trufflesecurity/trufflehog@v3.76.0
        with:
          path: .
          # scan only the diff since last commit to keep runtime small
          #   -- since the job runs on push to main, we compare to previous main commit
          base: ${{ github.event.before }}
          head: ${{ github.sha }}
          extra_args: --no-update --only-verified

      # Mark the job failed if secrets were found
      - name: Fail on findings
        run: |
          if [ -f trufflehog-results.json ]; then
            echo "🚨 Potential secrets detected"
            cat trufflehog-results.json
            exit 1
          fi```
(If coverage is slow, mark the step as continue-on-error: true.)



---

## Appendix 3: Optional Design Freeze Action
<!-- .github/workflows/design-freeze.yml -->
```yaml
name: Block merge on design-freeze label

on:
  pull_request:
    types: [labeled, unlabeled, opened, synchronize]

jobs:
  guard:
    if: contains(github.event.pull_request.labels.*.name, 'design-freeze')
    runs-on: ubuntu-latest
    steps:
      - name: Fail when design-freeze present
        run: |
          echo "PR is under design-freeze. Remove the label before merging."
          exit 1
```

---

## Appendix 4: Scaffold Workflow (repetitive workflow checklist for agents)
<!-- agent-context/workflow.md -->
```markdown
# workflow.md  
_Per-session operational steps for agents_

---

### 1 Start-of-session

1. **Load context**  
   * Read `agent-context/session-log.md` & all spec files.
2. **Ensure artefacts**  
   * If mandatory files are missing → create with `v1` headers.
3. **Version bump**  
   * Increment session integer (`vN → vN+1`).
   * Start log entry:

     ```markdown
     ---
     session: v{N+1}
     timestamp: <ISO 8601 America/Toronto>
     agent: OpenAI-Codex
     ---
     _Objective: <natural-language summary of user request>_
     ```

---

### 2 Design-spec review & update

1. Update relevant spec files pertinent to the session objective.  
2. Bump their internal version to match `v{N+1}`.  
3. Do **not** touch spec files unrelated to this session.

---

### 3 Coding phase

* Follow coding conventions from **AGENTS.md**.  
* Detect hard-coded secrets → Refactor to env vars + update `.env.example`.  
* If any new logic →  Add or modify unit tests accordingly.  
* If any SQL changes are required or proposed →  Generate `agent-context/sql-diff-v{NN}.sql` with both `-- UP:` and `-- DOWN:` blocks.

---

### 4 Post-code automated tasks
Before committing, run `pnpm test` from the repository root. The script currently prints a placeholder message but acts as a sanity check.

```bash
pnpm env:sync           # update .env.example
pnpm spec:lint          # warn if spec versions mismatched
pnpm lint               # ESLint + solhint (warnings allowed)
pnpm test               # forge test
```
(Coverage is optional; run pnpm coverage when appropriate.)


---

### 5 End-of-session log & reflection

Append to the **same** session block in `session-log.md`:

```markdown
#### Summary
- Key changes (files, components, tests)
- SQL patch: sql-diff-v{N} (if any)
#### Reflections. List:
- At least one confusion encountered
- At least one improvement idea for next session
- At least one test that could be improved next session
```

Push commit (use a scoped, descriptive message).

---

### 6 Optional artefact creation rules

| Optional file       | When to create                               | Header template        |
| -----------------   | -------------------------------------------- | -------------------    |
| `user-stories.md`   | When personas or acceptance tests are useful | `# User Stories v1`    |
| `app-context.md`    | When framing problem/value is helpful        | `# App Context v1`     |
| `sql-diff-v{N}.sql` | First time SQL diff is needed                | `-- SQL Diff v0->v1`   |

Each new file must be referenced in `agent-context/spec.md` under “Related Docs”.

---

*End of workflow\.md*

```
