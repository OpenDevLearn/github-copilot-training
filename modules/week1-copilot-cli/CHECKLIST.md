# Week 1: Copilot + CLI — Review Checklist

## Per-Lab Review

### Lab 1: CLI Automation
- [ ] CLI commands execute without manual shell editing after prompt
- [ ] Refactored code passes existing test suite
- [ ] Diff is clean (no unintended changes outside target modules)
- [ ] Evidence: CLI transcript committed to `evidence/`

### Lab 2: Natural-Language Git
- [ ] Branch name follows org convention (`feat/`, `fix/`, etc.)
- [ ] Commit message is semantic and references intent
- [ ] PR body is informative (not generic boilerplate)
- [ ] No credentials in any committed artifact

### Lab 3: Batch Refactor
- [ ] All target files processed (none silently skipped)
- [ ] Human approval log shows decision for each file
- [ ] Checkpoint/resume tested (stop mid-batch, resume, complete)
- [ ] Rollback tested (tag exists, `git reset` works)
- [ ] Batch log includes correlation ID

## Cross-Cutting Checks
- [ ] No secrets in prompt templates or logs (`detect-secrets` scan passes)
- [ ] All evidence artifacts are repo-committed
- [ ] Mermaid diagram documents the batch workflow
- [ ] Prompt templates stored in `modules/week1-copilot-cli/examples/prompts/`

## Pitfalls to Watch
| Pitfall | Why It Matters | Detection |
|---------|---------------|-----------|
| No dry-run before apply | Irreversible changes | Review batch script for `--dry-run` step |
| Tokens in prompts | Security breach | `detect-secrets` scan |
| No checkpoint | Lost progress on failure | Test by killing mid-batch |
| Overly broad file glob | Unintended files modified | Review `find` command filters |
| Missing rollback tag | Can't undo batch | Check for `pre-refactor-*` Git tag |
