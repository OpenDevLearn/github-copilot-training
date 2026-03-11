# Week 3: Skills — Review Checklist

## Per-Lab Review

### Lab 1: Skill Authoring
- [ ] `SKILL.md` has valid YAML frontmatter (name, version, parameters, dependencies)
- [ ] Parameters have type, required flag, and description
- [ ] Procedure is step-by-step and unambiguous
- [ ] Output schema is JSON-valid and documented
- [ ] Failure modes are listed with recovery strategy
- [ ] Test cases included (at least 5: happy path, edge cases, failure)

### Lab 2: Skill Composition
- [ ] Chain definition declares data flow between steps
- [ ] Conditional execution logic is explicit
- [ ] Intermediate outputs are named and typed
- [ ] Chain halts gracefully on step failure (returns partial results)
- [ ] Mermaid diagram shows chain flow

### Lab 3: Skill Versioning
- [ ] All skills use semver (MAJOR.MINOR.PATCH)
- [ ] CHANGELOG exists for each skill
- [ ] Dependencies use version constraints (`^`, `~`)
- [ ] DAG validator script passes (zero cycles)
- [ ] Breaking change example documented with migration path

## Cross-Cutting Checks
- [ ] All skills are stateless and idempotent
- [ ] No hardcoded paths or environment assumptions
- [ ] Audit logging includes skill name, version, and correlation ID
- [ ] Skills degrade gracefully when dependencies unavailable
- [ ] Dependency graph documented as Mermaid diagram

## Pitfalls to Watch
| Pitfall | Why It Matters | Detection |
|---------|---------------|-----------|
| God skill (does everything) | Unmaintainable, untestable | Skill has >3 parameters or >10 procedure steps |
| Implicit dependencies | Breaks silently when dep changes | DAG validator doesn't see the link |
| No versioning | Consumers break on changes | Check for semver in frontmatter |
| Hardcoded paths | Fails in different repos | Grep for absolute paths in skill files |
| Missing output schema | Consumers can't validate results | Check for `Output Schema` section |
