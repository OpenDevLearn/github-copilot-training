# Lab 3: Multi-Step Planning with Checkpoints

## Objective
Design and execute a multi-step agent workflow (5 steps) with explicit approval gates, structured checkpoint data, and complete audit trail.

## Prerequisites
- Completed Labs 1 and 2
- Sample codebase with known security issues for the agent to review

## Setup
```bash
# Create a sample codebase with intentional security issues
mkdir -p src/api src/auth src/db

cat > src/api/handler.py << 'PYEOF'
import subprocess
from flask import request, jsonify

def execute_command():
    cmd = request.args.get("cmd")
    result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
    return jsonify({"output": result.stdout})

def search_users():
    query = request.args.get("q")
    sql = f"SELECT * FROM users WHERE name LIKE '%{query}%'"
    return execute_query(sql)
PYEOF

cat > src/auth/login.py << 'PYEOF'
import hashlib

SECRET_KEY = "hardcoded-secret-key-12345"

def hash_password(password):
    return hashlib.md5(password.encode()).hexdigest()

def verify_token(token):
    # No expiry check
    return decode_jwt(token, SECRET_KEY)
PYEOF

cat > src/db/connection.py << 'PYEOF'
DB_PASSWORD = "production-password-123"

def get_connection():
    return connect(
        host="db.internal.example.com",
        password=DB_PASSWORD,
        ssl=False
    )
PYEOF
```

## Tasks

### Task 1: Define the 5-step plan

Create `agent-plans/security-review.md`:

| Step | Action | Gate Type | Output |
|------|--------|-----------|--------|
| 1 | Reconnaissance — inventory files, identify frameworks | Auto | File inventory |
| 2 | Pattern scan — OWASP Top 10 pattern matching | Auto | Preliminary findings |
| 3 | Threat modeling — STRIDE analysis | **Human review** | Threat model |
| 4 | Deep analysis — exploitability assessment | **Human approval** | Detailed findings |
| 5 | Report generation — prioritized remediations | **Sign-off** | Final report |

**Acceptance:** Plan document exists with all 5 steps, gate types, and outputs defined.

### Task 2: Execute Steps 1-2 (auto gates)
Invoke the security-auditor agent with the plan. Steps 1 and 2 should execute automatically.

**Acceptance:**
- Step 1 produces file inventory (3 files, Python, Flask framework detected)
- Step 2 identifies at least 5 issues:
  - Command injection in `handler.py`
  - SQL injection in `handler.py`
  - Hardcoded secret in `login.py`
  - Weak hashing (MD5) in `login.py`
  - Hardcoded DB password in `connection.py`

### Task 3: Execute Step 3 (human review gate)
Agent should STOP after Step 2 and present findings for human review.

Review the threat model:
- Verify STRIDE categories are correct
- Confirm no false positives
- Approve to continue

**Acceptance:**
- Agent paused at Step 3 gate (did not auto-proceed)
- Threat model includes STRIDE mapping for each finding
- Human review decision logged with timestamp

### Task 4: Execute Steps 4-5 (human approval gates)
After approval, agent proceeds to deep analysis and report generation.

**Acceptance:**
- Step 4: Each finding has exploitability assessment and impact score
- Step 5: Final report in structured format with prioritized remediations
- Each gate has human decision logged

### Task 5: Validate the audit trail
```bash
# Verify complete audit trail
cat evidence/lab3-audit-trail.log

# Expected structure:
# [timestamp] SESSION_START agt-{uuid}
# [timestamp] STEP_1_START agt-{uuid}-step-001
# [timestamp] TOOL_CALL agt-{uuid}-step-001-tc-001 list_dir src/
# ... (tool calls)
# [timestamp] STEP_1_COMPLETE output_hash={sha256}
# [timestamp] STEP_2_START agt-{uuid}-step-002
# ... (tool calls)
# [timestamp] STEP_2_COMPLETE output_hash={sha256}
# [timestamp] GATE_3 HUMAN_REVIEW_REQUIRED
# [timestamp] GATE_3 HUMAN_APPROVED reviewer={username}
# ... (steps 4-5)
# [timestamp] SESSION_COMPLETE agt-{uuid}
```

**Acceptance:**
- All 5 steps logged with start/complete markers
- All 3 human gates logged with decision and reviewer
- Correlation IDs are consistent throughout
- No secrets appear in logs

## Evidence to Commit
- [ ] `agent-plans/security-review.md` — 5-step plan
- [ ] `evidence/lab3-step1-inventory.md` — File inventory
- [ ] `evidence/lab3-step2-findings.md` — Preliminary findings
- [ ] `evidence/lab3-step3-threat-model.md` — Threat model (STRIDE)
- [ ] `evidence/lab3-step4-deep-analysis.md` — Detailed findings
- [ ] `evidence/lab3-step5-final-report.md` — Prioritized report
- [ ] `evidence/lab3-audit-trail.log` — Complete audit trail
- [ ] `docs/security-review-workflow.mmd` — Mermaid diagram of workflow

## Verification
```bash
# Count evidence files
ls evidence/lab3-* | wc -l  # Should be ≥7

# Verify all steps represented
for step in step1 step2 step3 step4 step5 audit; do
    ls evidence/lab3-*$step* 2>/dev/null && echo "STEP $step: PRESENT" || echo "STEP $step: MISSING"
done

# Verify no secrets in audit trail
grep -iE "(password|secret|key|token)=[^R]" evidence/lab3-audit-trail.log && echo "REDACTION: FAIL" || echo "REDACTION: PASS"
```
