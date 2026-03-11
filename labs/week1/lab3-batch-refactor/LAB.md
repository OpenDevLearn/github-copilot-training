# Lab 3: Batch Refactor with Prompt-in-the-Loop

## Objective
Execute a batch refactor across 10+ Python files using Copilot CLI with human approval gates, checkpoint/resume, and audit logging.

## Prerequisites
- Completed Labs 1 and 2
- A Python repo with 10+ files needing refactoring (type annotations, docstrings, or naming)
- `gh` CLI with Copilot extension

## Setup
```bash
# Generate sample files for batch processing
mkdir -p labs/week1/lab3-batch-refactor/workspace/src
cd labs/week1/lab3-batch-refactor/workspace

for i in $(seq 1 12); do
cat > "src/service_${i}.py" << EOF
def process_data_${i}(input_data, config, retries):
    """Process data for service ${i}."""
    for attempt in range(retries):
        try:
            result = transform(input_data, config)
            return result
        except Exception as e:
            if attempt == retries - 1:
                raise
    return None

def validate_input_${i}(data, schema):
    if not isinstance(data, dict):
        return False
    for key in schema:
        if key not in data:
            return False
    return True
EOF
done

# Initialize git
git init && git add . && git commit -m "initial: 12 services without type annotations"
```

## Tasks

### Task 1: Create the batch script
Write a batch refactor script that:
1. Collects all `.py` files in `src/`
2. For each file: uses Copilot CLI to explain what annotations are needed
3. Asks for human approval before applying
4. Supports checkpoint/resume (writes progress to `.checkpoint`)
5. Logs every decision to `evidence/batch-log-{timestamp}.log`
6. Creates a Git tag `pre-refactor-{timestamp}` before starting

**Acceptance:** Script exists and handles all 5 requirements.

### Task 2: Execute with dry-run
```bash
# Run the batch script in dry-run mode first
bash batch-refactor.sh --dry-run
```
**Acceptance:** Dry-run completes for all 12 files. No files modified. Log shows planned changes.

### Task 3: Execute with approval gates
```bash
# Run for real — approve some, skip some, quit mid-batch
bash batch-refactor.sh
# Approve files 1-5, skip file 6, quit at file 7
```
**Acceptance:**
- Files 1-5 have type annotations
- File 6 is unchanged
- Checkpoint file shows index 7
- Log shows all decisions

### Task 4: Resume from checkpoint
```bash
# Resume the batch from where we left off
bash batch-refactor.sh
# Approve remaining files
```
**Acceptance:**
- Processing starts at file 7 (not file 1)
- All remaining files processed
- Checkpoint file removed on completion

### Task 5: Test rollback
```bash
# Verify rollback works
git diff --stat pre-refactor-* HEAD
git stash  # or reset to tag to verify
git stash pop  # restore
```
**Acceptance:** `git reset --hard pre-refactor-{tag}` restores all files to pre-refactor state.

### Task 6: Validate results
```bash
# Run mypy across all processed files
mypy src/ --strict 2>&1 | tee evidence/lab3-mypy-results.txt

# Count processed vs total
echo "Processed: $(grep 'APPLIED' evidence/batch-log-*.log | wc -l) / 12"
```
**Acceptance:** 100% of approved files pass `mypy --strict`. Batch log accounts for all 12 files.

## Evidence to Commit
- [ ] `batch-refactor.sh` — The batch script
- [ ] `evidence/batch-log-*.log` — Complete batch log with correlation ID
- [ ] `evidence/lab3-mypy-results.txt` — mypy validation
- [ ] `evidence/lab3-rollback-test.txt` — Rollback verification output

## Verification
```bash
# Comprehensive check
test -f batch-refactor.sh && echo "SCRIPT: PASS" || echo "SCRIPT: FAIL"
ls evidence/batch-log-*.log && echo "LOG: PASS" || echo "LOG: FAIL"
grep -c "APPLIED\|SKIPPED" evidence/batch-log-*.log | grep -q "12" && echo "COVERAGE: PASS" || echo "COVERAGE: FAIL"
git tag | grep -q "pre-refactor" && echo "ROLLBACK TAG: PASS" || echo "ROLLBACK TAG: FAIL"
! test -f .checkpoint && echo "CHECKPOINT CLEANUP: PASS" || echo "CHECKPOINT CLEANUP: FAIL"
```
