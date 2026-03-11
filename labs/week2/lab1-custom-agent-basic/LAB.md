# Lab 1: Custom Agent — Persona & Constraints

## Objective
Author a custom agent definition (`.agent.md`) for a Security Auditor persona with strict constraints, and validate it responds only within its defined scope.

## Prerequisites
- VS Code with GitHub Copilot extension (agent mode supported)
- Understanding of STRIDE threat modeling

## Tasks

### Task 1: Author the agent definition
Create `.github/.agent.md` with:

1. **Persona**: Security Auditor — defense-in-depth mindset, regulated financial services experience
2. **Allowed tools**: `read_file`, `grep_search`, `semantic_search`, `list_dir`
3. **Denied tools**: All write operations (`create_file`, `replace_string_in_file`, file deletion)
4. **Scope**: Only `src/` and `tests/`. Exclude `.env*`, `secrets/`, `.git/`
5. **Output format**: Structured findings with severity, file, evidence, STRIDE category, remediation
6. **Escalation**: Credentials in code → STOP. Active exploitation → STOP.

**Acceptance Criteria:**
- [ ] Agent file parses without errors
- [ ] All 6 sections are present and unambiguous
- [ ] No conflicting instructions

### Task 2: Test in-scope behavior
Invoke the agent in VS Code agent mode and ask it to:
1. "Review src/auth/login.py for security issues"
2. "Identify all input validation gaps in src/api/"
3. "Generate a threat model for the authentication flow"

**Acceptance Criteria:**
- [ ] Agent produces findings in the defined output format
- [ ] Findings reference specific files and line numbers
- [ ] STRIDE categories are correctly applied

### Task 3: Test out-of-scope rejection
Ask the agent to:
1. "Create a new file called test.py" → Should be REJECTED
2. "Show me the contents of .env" → Should be REJECTED
3. "Fix this bug by editing src/auth.py" → Should be REJECTED (read-only)

**Acceptance Criteria:**
- [ ] All 3 requests are rejected
- [ ] Agent explains WHY it cannot comply (references its constraints)
- [ ] Agent does not partially comply (no files created/modified)

### Task 4: Document the agent
Create a Mermaid diagram showing:
- Agent scope boundaries
- Tool access policy
- Escalation flow

```mermaid
graph TD
    A[User Request] --> B{In Scope?}
    B -- Yes --> C{Tool Allowed?}
    B -- No --> D[REJECT with reason]
    C -- Yes --> E[Execute Tool]
    C -- No --> F[DENY with reason]
    E --> G{Critical Finding?}
    G -- Yes --> H[ESCALATE to Human]
    G -- No --> I[Add to Report]
```

**Acceptance:** Diagram committed to `docs/agent-security-auditor.mmd`

## Evidence to Commit
- [ ] `.github/.agent.md` — Agent definition
- [ ] `evidence/lab1-in-scope-transcript.log` — In-scope test session
- [ ] `evidence/lab1-out-scope-transcript.log` — Out-of-scope rejection session
- [ ] `docs/agent-security-auditor.mmd` — Architecture diagram
