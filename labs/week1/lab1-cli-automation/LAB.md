# Lab 1: CLI Automation — Local Refactoring

## Objective
Use Copilot CLI to refactor a Python project across 3 modules without manual code editing.

## Prerequisites
- `gh` CLI with Copilot extension installed
- A Python project with at least 3 modules containing functions without type annotations

## Setup
```bash
# Clone the practice repo (or use your own)
mkdir -p labs/week1/lab1-cli-automation/workspace
cd labs/week1/lab1-cli-automation/workspace

# Create sample modules if needed
cat > module_a.py << 'EOF'
def calculate_discount(price, rate, min_price):
    result = price * (1 - rate)
    return max(result, min_price)

def format_currency(amount, currency, locale):
    symbols = {"USD": "$", "EUR": "€", "GBP": "£"}
    return f"{symbols.get(currency, currency)}{amount:.2f}"
EOF

cat > module_b.py << 'EOF'
import json

def parse_config(filepath, defaults):
    with open(filepath) as f:
        config = json.load(f)
    merged = {**defaults, **config}
    return merged

def validate_config(config, schema):
    errors = []
    for key, expected_type in schema.items():
        if key in config and not isinstance(config[key], expected_type):
            errors.append(f"{key}: expected {expected_type.__name__}")
    return errors
EOF

cat > module_c.py << 'EOF'
from datetime import datetime, timedelta

def calculate_deadline(start_date, business_days, holidays):
    current = start_date
    days_added = 0
    while days_added < business_days:
        current += timedelta(days=1)
        if current.weekday() < 5 and current not in holidays:
            days_added += 1
    return current

def is_overdue(deadline, current_time):
    return current_time > deadline
EOF
```

## Tasks

### Task 1: Explain before modifying
```bash
# Use Copilot to analyze each module before touching it
gh copilot explain "What are the function signatures in module_a.py and what types would be appropriate for each parameter and return value?"
```
**Acceptance:** Copilot identifies all functions and suggests reasonable types.

### Task 2: Apply type annotations via CLI
```bash
# For each module, use Copilot CLI to add type annotations
gh copilot suggest "Add PEP 484 type annotations to all functions in module_a.py. Use built-in types for Python 3.10+. Do not change logic."
```
**Acceptance:** Type annotations added to all 6 functions across 3 modules.

### Task 3: Validate
```bash
# Run mypy to validate type annotations
pip install mypy
mypy module_a.py module_b.py module_c.py --strict
```
**Acceptance:** `mypy --strict` passes with zero errors.

### Task 4: Commit with evidence
```bash
# Commit with audit trail
git add .
git commit -m "feat(types): add PEP 484 annotations to modules A/B/C via Copilot CLI

Copilot-assisted: true
Session: cli-$(uuidgen | tr '[:upper:]' '[:lower:]')"
```
**Acceptance:** Commit includes `Copilot-assisted: true` tag.

## Evidence to Commit
- [ ] `evidence/lab1-transcript.log` — CLI session transcript
- [ ] `evidence/lab1-mypy-output.txt` — mypy validation results
- [ ] Modified `module_a.py`, `module_b.py`, `module_c.py` with annotations

## Verification
```bash
# Verify all acceptance criteria
mypy module_a.py module_b.py module_c.py --strict && echo "PASS" || echo "FAIL"
git log -1 --format="%B" | grep -q "Copilot-assisted" && echo "AUDIT: PASS" || echo "AUDIT: FAIL"
ls evidence/lab1-*.{log,txt} 2>/dev/null && echo "EVIDENCE: PASS" || echo "EVIDENCE: FAIL"
```
