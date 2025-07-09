### ☑️  PR Checklist

- [ ] **Session log**:
      - [ ] Version bumped
      - [ ] Appended session summary at top of `agent-context/session-log.md` for **v{N}**
      - [ ] Reflections section completed, including one confusion and one suggestion to the next agent
- [ ] **Spec files**: Relevant spec(s) updated and bumped to version **v{N}**
- [ ] **Agent instructions**: AGENTS.md updated with relevant changes, if any
- [ ] **Lint**: `pnpm lint` passes *or* ESLint failure noted in commit
- [ ] **Tests**:
      - [ ] New or affected logic covered by unit tests
      - [ ] E2E passes locally
- [ ] **Environment**: `.env.example` updated with any new variable names
- [ ] **SQL changes,** if any:  
      - [ ] Matching `sql-diff-v{N}` with **UP / DOWN** blocks committed  
      - [ ] I have (or will) apply the patch in Supabase _or_ open a follow‑up task

_If this PR is subject to a **design-freeze** label, remove the label before merging._
