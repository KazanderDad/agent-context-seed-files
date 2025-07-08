---
description: Authorative per-session operational steps for agents coding in this repository
alwaysApply: true
---

# workflow.md
This workflow follows three sections 1. Start of session, 2. Coding session, 3. Lint and test. 4. End of session. 

---

## 1. Start-of-session

---

### Load context versioning context
  * Read the last entry or last few entries in `agent-context/session-log.md`

---

### **Version bump**  
  * If you are on a a new feature branch (or on Main): Increment session integer (e.g. from v1.0 to v2.0).
  * If you are on a the same feature branch as previous session: Increment session decimal (e.g. from v2.0 to v2.1).

---

### Start session log entry
Add a new entry in or around row 14, following this template:

```markdown
## Session vN.N
- timestamp: **<ISO 8601 America/Toronto>**
- agent: **<agent-name>** (e.g. OpenAI‑Codex)
- session name: **<session-name>**
- branch: **<Branch>**

Session objective: <natural-langauage summary of user request>
```

---

### Load context
Read any spec files relevant to your sesssion objective

### Optional artefact creation
If mandatory files are missing → create with `v1` headers

| Optional file | When to create | Header template |
| --- | --- | --- |
| user-stories.md | When personas or acceptance tests are useful | # User Stories v1 |
| app-context.md | When framing problem/value is helpful | # App Context v1 |
| sql-diff-v{N}.sql | First time SQL diff is needed | -- SQL Diff v0->v1 |

Each new file must be referenced in agent-context/spec.md under “Related Docs”.

---

## Design-spec review & update

1. Update relevant spec files pertinent to the session objective.  
2. Bump their internal version to match `v{N+1}`.  
3. Do **not** touch spec files unrelated to this session.

---

## 2. Coding phase
Follow coding conventions from **AGENTS.md**, and also:

* Detect hard-coded secrets → Refactor to env vars + update `.env.example`.  
* If any new logic →  Add or modify unit tests accordingly.  
* If any SQL changes are required or proposed →  Generate new file `agent-context/sql-diff-v{NN}.sql` with both `-- UP:` and `-- DOWN:` blocks.

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

## End of Session

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

### Push commit
Use a scoped, descriptive message. 
This step is optional, as it may be up to the user to accept before committing or creating a pull request.
