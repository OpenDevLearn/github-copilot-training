# Week 1: Copilot + CLI for Automation — Deep Dive

---

## Mental Model

Copilot CLI is a **natural-language shell interface** that translates intent into executable commands. Think of it as a composable automation layer sitting between human intent and shell execution.

```
┌─────────────┐     ┌──────────────┐     ┌─────────────┐
│  Natural     │ ──▶ │  Copilot CLI │ ──▶ │  Shell Cmd  │
│  Language    │     │  (interpret)  │     │  (execute)  │
└─────────────┘     └──────────────┘     └─────────────┘
        │                                        │
        ▼                                        ▼
   Prompt Template                        Audit Log + Diff
```

**Key principle:** CLI automation replaces manual shell workflows. Every action is auditable, repeatable, and Git-traceable.


### When NOT to Use
- ❌ Real-time production operations (use proper runbooks/tooling)
- ❌ Operations requiring transactional guarantees across systems
- ❌ When deterministic scripts already exist and are well-tested
- ❌ Sensitive operations that need formal change management beyond Git

---

**References**
- [About copilot CLI](https://docs.github.com/en/copilot/concepts/agents/copilot-cli/about-copilot-cli)
- [allowing-tools-to-be-used-without-manual-approval](https://docs.github.com/en/copilot/concepts/agents/copilot-cli/about-copilot-cli#allowing-tools-to-be-used-without-manual-approval)



## Implementation Patterns

### Pattern 1: Natural-Language Git Commands

```bash
# Branch creation from natural language
gh copilot suggest "create a new branch called feat/add-auth-middleware from main"

# Commit with semantic message
gh copilot suggest "stage all Python files in src/auth/ and commit with message 'feat(auth): add JWT middleware with role-based claims'"

# PR creation with context
gh copilot suggest "create a pull request from feat/add-auth-middleware to main with title 'Add JWT auth middleware' and body describing the RBAC implementation"
```

### Pattern 2: Batch Refactor with Prompt-in-the-Loop

```bash
#!/usr/bin/env bash
# batch-refactor.sh — Batch rename + type annotation via Copilot CLI
# Requires: gh copilot extension

set -euo pipefail

TARGET_DIR="src/"
LOG_FILE="evidence/batch-refactor-$(date +%Y%m%d-%H%M%S).log"
CHECKPOINT_FILE=".refactor-checkpoint"

# Resume from checkpoint if exists
START_INDEX=0
if [[ -f "$CHECKPOINT_FILE" ]]; then
    START_INDEX=$(cat "$CHECKPOINT_FILE")
    echo "Resuming from file index $START_INDEX"
fi

# Collect target files
mapfile -t FILES < <(find "$TARGET_DIR" -name "*.py" -type f | sort)
TOTAL=${#FILES[@]}

echo "Processing $TOTAL files (starting at $START_INDEX)" | tee -a "$LOG_FILE"

for ((i=START_INDEX; i<TOTAL; i++)); do
    FILE="${FILES[$i]}"
    echo "[$((i+1))/$TOTAL] Processing: $FILE" | tee -a "$LOG_FILE"

    # Dry-run first
    echo "--- DRY RUN for $FILE ---" >> "$LOG_FILE"
    gh copilot explain "Add type annotations to all function signatures in $FILE following PEP 484" 2>&1 | tee -a "$LOG_FILE"

    # Human approval gate
    read -rp "Apply changes to $FILE? [y/N/s(skip)/q(quit)] " RESPONSE
    case "$RESPONSE" in
        y|Y)
            gh copilot suggest "Add type annotations to all function signatures in $FILE" 2>&1 | tee -a "$LOG_FILE"
            echo "$FILE: APPLIED" >> "$LOG_FILE"
            ;;
        s|S)
            echo "$FILE: SKIPPED" >> "$LOG_FILE"
            ;;
        q|Q)
            echo "$i" > "$CHECKPOINT_FILE"
            echo "Checkpointed at index $i. Run again to resume." | tee -a "$LOG_FILE"
            exit 0
            ;;
        *)
            echo "$FILE: SKIPPED (default)" >> "$LOG_FILE"
            ;;
    esac

    # Save checkpoint
    echo "$((i+1))" > "$CHECKPOINT_FILE"
done

# Cleanup checkpoint on completion
rm -f "$CHECKPOINT_FILE"
echo "Batch complete. Log: $LOG_FILE"
```

### Pattern 3: Scripted Prompt Templates

```bash
# prompts/add-type-annotations.txt
# Reusable prompt template for batch operations
# Variables: {{FILE_PATH}}, {{STYLE_GUIDE}}

You are a senior Python engineer enforcing PEP 484 type annotations.

Task: Add type annotations to all function signatures in {{FILE_PATH}}.

Rules:
1. Use built-in types (list, dict, tuple) for Python 3.9+
2. Use Optional[] for nullable parameters
3. Use Union[] sparingly — prefer overloads
4. Return type required for all functions
5. Follow project style: {{STYLE_GUIDE}}

Do NOT:
- Change function logic
- Add runtime type checks
- Modify docstrings
- Import from typing if built-in equivalent exists
```

---

## Governance & Security Controls

| Control | Implementation |
|---------|---------------|
| **RBAC** | CLI operations scoped to user's Git permissions. No elevation. |
| **Data boundaries** | Prompt templates must not contain secrets. Pre-scan with `detect-secrets`. |
| **Audit hooks** | All CLI sessions logged to `evidence/` with timestamp, user, and diff. |
| **Approval gates** | Batch operations require per-file human confirmation (see Pattern 2). |
| **Rollback** | Every batch creates a Git tag `pre-refactor-{timestamp}` before execution. |

---

## Observability

### What to Log
- CLI invocation timestamp and user
- Natural-language prompt (sanitized)
- Generated command or code diff
- Human decision (apply/skip/quit)
- Execution result (success/failure/error)

### Correlation & Redaction
```bash
# Generate correlation ID for batch session
CORRELATION_ID="cli-$(uuidgen | tr '[:upper:]' '[:lower:]')"
echo "Session: $CORRELATION_ID" >> "$LOG_FILE"

# Redaction: strip tokens, keys, and internal hostnames
sed -i 's/ghp_[A-Za-z0-9]\{36\}/[REDACTED_TOKEN]/g' "$LOG_FILE"
sed -i 's/[a-zA-Z0-9._%+-]\+@[a-zA-Z0-9.-]\+\.[a-zA-Z]\{2,\}/[REDACTED_EMAIL]/g' "$LOG_FILE"
```

---

## Test Strategy

| Test Type | What to Validate |
|-----------|-----------------|
| **Replay tests** | Record CLI session → replay → assert same output for same input |
| **Adversarial** | Inject prompt-manipulation in file names/content → assert CLI doesn't execute |
| **Idempotency** | Run batch refactor twice → assert no additional changes on second run |
| **Rollback** | After batch, `git reset --hard pre-refactor-tag` restores original state |
| **Boundary** | Test with 0 files, 1 file, 1000 files — assert checkpoint/resume works |

---

## Performance Considerations

- **Rate limits**: Copilot CLI has API rate limits. Batch operations should include backoff.
- **Large repos**: Use `find` with filters to limit scope. Don't process `node_modules/`, `.git/`, etc.
- **Network latency**: Each CLI call requires a round-trip. Batch prompts where possible.

## Failure Modes

| Failure | Detection | Recovery |
|---------|-----------|----------|
| API timeout | CLI returns non-zero exit code | Retry with exponential backoff (max 3) |
| Malformed suggestion | Output fails schema validation | Log and skip file, continue batch |
| Git conflict | `git status` shows conflicts | Checkpoint and alert human |
| Rate limit exceeded | HTTP 429 in CLI output | Pause batch, resume after cooldown |

---

## Rollout Playbook

1. **Pilot (1 dev, 1 repo)**: Run all 3 labs. Collect feedback on prompt quality and CLI ergonomics.
2. **Team trial (3-5 devs)**: Share prompt templates. Standardize logging format. Review evidence artifacts.
3. **Org rollout**: Publish prompt template library to internal repo. Define batch-operation guidelines. Monitor audit logs weekly.
4. **Governance**: Add CLI usage to developer onboarding. Include in code review checklist.
