---
description: Authoritative per-session workflow for agents contributing to this codebase. Defines the steps agents must follow to ensure consistent context loading, design versioning, secure coding, and test hygiene.
alwaysApply: true
---

# workflow.md
- This workflow follows steps: 1. Start of session, 2. Coding session, 3. Lint and test. 4. End of session. 
- Steps are mandatory unless marked optional. 
- Iteration is allowed and encouraged.

---

## 1. Start-of-session

---

### Session log and versioning context
- Read the last entry in `agent-context/session-log.md`
- Version bump:
  - If you are on a new feature branch (or on Main): Increment session integer (e.g. from v1.0 to v2.0).
  - If you are on the same feature branch as previous session: Increment session decimal (e.g. from v2.0 to v2.1).
- Start session log entry. Add a new entry in or around row 15, following this template:

```markdown
## Session vN.N
- timestamp: **<ISO 8601 America/Toronto>**
- agent: **<agent-name>** (e.g. OpenAI‑Codex)
- session name: **<session-name>**
- branch: **<Branch>**

Session objective: <natural-language summary of user request>
```

---

### Load context
- Read any context files which are already in the following folders and may be relevant to your session objective:
  - `agent-context/*.*`
  - `agent-instructions/*.*`
- Read any additional external instruction files in the below folder which may be relevant to your session objective:
  - `https://github.com/KazanderDad/agent-context-seed-files/tree/main/agent-instructions/*.*`
  - If these files are not already copied into the repo, they may be referenced directly from GitHub. However, if the agent determines they will be edited locally, they MUST first be copied from the seed repo into `agent-instructions/`.

FYI, these are the folders you can expect to find in the [external repo](https://github.com/KazanderDad/agent-context-seed-files/tree/main/agent-instructions/):
| External instruction file | When to use |
| --- | --- |
| supabase-bootstrap.md | When initializing supabase into a repo |
| postgres-sql-style-guide.md | Anytime supabase is used in the repo  |
| supabase-declarative-schema.md | Anytime supabase is used in the repo  |
| supabase-edge-functions.md | When you need to write or modify edge functions in supabase |
| supabase-functions.md | When you need to write or modify database functions in supabase |
| supabase-migrations.md | When the ci.yml has been or should be initialized with supabase migrations  |
| supabase-rls-policies.md | Anytime supabase is used in the repo |
| any other file | as per file content |

---

### Artefact creation (optional)
If any optional files (see table below) are missing but should be added based on your session objectives, then:
- Copy seed files and templates from `https://github.com/KazanderDad/agent-context-seed-files/` into your repo.
- Do not create these files from scratch (unless you need to create a file which is not available as a template or seed file).
- Source directory: `https://github.com/KazanderDad/agent-context-seed-files/tree/main/agent-context-optional`
- Target directory: `agent-context/`

FYI, these are the folders you can expect to find in the external repo:
| Optional file | When to copy to local repo |
| --- | --- |
| app-context.md | When framing problem/value is helpful |
| user-stories.md | When personas or acceptance tests are useful |
| functional-spec.md | When framing problem/value is helpful |
| any other file | as per file content |


After adding files:
- Each new file must also be referenced in agent-context/spec.md under “Related Docs” -> add
- Exception: SQL diff files (see step 4 below)

---

### Design-spec review & update

1. Update relevant spec files pertinent to the session objective.  
2. Bump their internal version to match `v{N+1}`.  
3. Do **not** touch specfiles unrelated to this session. If in doubt, err on the side of leaving specs untouched. If changes seem necessary outside your scope, leave a note in the session log or create a placeholder entry (e.g. TODO(v3.1): expand user-stories.md to include mobile flow).

---

## 2. Coding phase
Follow coding conventions from **AGENTS.md**, and also:

* If any new logic →  Add or modify unit tests accordingly.  
* If any SQL changes are required or proposed →  Generate new file `agent-context/sql-diff-v{NN}.sql` with both `-- UP:` and `-- DOWN:` blocks.
* Sanitize inputs and outputs for any external-facing components (e.g. API handlers, forms).

---

## 3. Lint and Test

### Post-code automated tasks
Before committing, run `pnpm test` from the repository root. The script currently prints a placeholder message but acts as a sanity check.

```bash
pnpm env:sync           # update .env.example
pnpm spec:lint          # warn if spec versions mismatched
pnpm lint               # ESLint + solhint (warnings allowed)
pnpm test               # forge test
```
(Coverage is optional; run pnpm coverage when appropriate.)

If tests reveal errors or warnings, then iterate back to the coding phase and improve the code until the results are satisfactory.

---

## 4. End of Session

### Create SQL diff file (optional)
If the coding session resulted in new tables or columns, or in requirement to update the existing SQL in supabase, then create a new file with the required changes (e.g a Make Table statement)

File name pattern: agent-context/sql-diff-v{N}.sql

File header template:
```markdown
- Created as part of session v{N}
- Apply to SQL Schema v{X} (where typically X = N-1 but sometimes applies to earlier version, for example if changes to N-1 were never implemented)
- Updated: 2025-06-26T09:45:00-04:00
- Target: Supabase
```


### Update log & reflection
Append to the new session block in session-log.md
- All key changes
- At least one confusion encountered
- At least one improvement idea for next session
- At least one test that could be improved next session

Use this template pattern:

```markdown
Key changes (files, components, tests):
- <Actions taken>
- <SQL patch: sql-diff-v{N}> (if any)

Reflections:
- Confusion(s) encountered
- Suggested next step(s)
- Suggested test(s) to add or improve on
    _Objective: <natural-language summary of user request>_
```

---

### Push commit (optional)
Use a scoped, descriptive message. 
This step is optional, as it may be up to the user to accept before committing or creating a pull request.
