# Lab 2: Natural-Language Git Commands

## Objective
Use Copilot CLI exclusively (no manual `git` commands) to perform a complete Git workflow: branch, commit, and PR creation.

## Prerequisites
- `gh` CLI with Copilot extension
- A Git repo with at least one commit on `main`
- GitHub remote configured

## Tasks

### Task 1: Create a feature branch
```bash
# Use natural language to create and switch to a new branch
gh copilot suggest "create a new git branch called feat/natural-lang-git from the main branch and switch to it"
```
**Acceptance:** Branch `feat/natural-lang-git` exists and is checked out.

### Task 2: Make meaningful changes
Create a small Python utility via Copilot CLI:
```bash
gh copilot suggest "create a Python file called utils/date_helpers.py with a function that calculates the number of business days between two dates, excluding weekends and a list of holidays"
```
**Acceptance:** File exists with correct function logic.

### Task 3: Stage and commit with semantic message
```bash
gh copilot suggest "stage the new file utils/date_helpers.py and commit with a conventional commit message describing a new utility for business day calculation"
```
**Acceptance Criteria:**
- Commit message follows conventional commits format (`feat(utils): ...`)
- Only the intended file is staged
- Message accurately describes the change

### Task 4: Create a Pull Request
```bash
gh copilot suggest "create a pull request from the current branch to main with a title describing the business day calculator and a body that lists what the function does, its parameters, and return type"
```
**Acceptance Criteria:**
- PR created successfully on GitHub
- Title is descriptive (not generic)
- Body contains parameter documentation
- PR is linked to the correct branches

### Task 5: Audit the workflow
```bash
# Capture the entire workflow for evidence
git log --oneline -5 > evidence/lab2-git-log.txt
gh pr view --json title,body,headRefName,baseRefName > evidence/lab2-pr-details.json
```
**Acceptance:** Evidence files committed showing complete workflow.

## Evidence to Commit
- [ ] `evidence/lab2-git-log.txt` — Git log showing branch and commit
- [ ] `evidence/lab2-pr-details.json` — PR metadata
- [ ] `evidence/lab2-cli-transcript.log` — Full CLI session transcript

## Verification
```bash
# Verify all criteria
git branch | grep -q "feat/natural-lang-git" && echo "BRANCH: PASS" || echo "BRANCH: FAIL"
test -f utils/date_helpers.py && echo "FILE: PASS" || echo "FILE: FAIL"
git log -1 --format="%s" | grep -qE "^(feat|fix|chore|docs)" && echo "COMMIT: PASS" || echo "COMMIT: FAIL"
gh pr view --json state -q '.state' | grep -q "OPEN" && echo "PR: PASS" || echo "PR: FAIL"
```
