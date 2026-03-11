# Week 2: Custom Agents — Review Checklist

## Per-Lab Review

### Lab 1: Agent Persona & Constraints
- [ ] `.agent.md` file has clear, unambiguous persona definition
- [ ] Constraints are specific (not vague like "be careful")
- [ ] Denial policy is explicit (lists what is denied, not just what is allowed)
- [ ] Agent responds only within defined scope
- [ ] Out-of-scope requests are rejected with explanation

### Lab 2: RBAC & Denial-by-Default
- [ ] Agent starts with zero tool access
- [ ] Each granted tool has justification
- [ ] Denied tools are tested (adversarial payloads confirm denial)
- [ ] Scope restrictions prevent access to sensitive paths
- [ ] Audit log captures denied tool attempts

### Lab 3: Multi-Step Planning
- [ ] Plan has ≤7 steps (no deep nesting)
- [ ] Each step has defined output and gate type (auto/human)
- [ ] Human gates cannot be bypassed
- [ ] Checkpoint data is structured and logged
- [ ] Final output matches defined schema

## Cross-Cutting Checks
- [ ] Adversarial injection tests pass (persona maintained under attack)
- [ ] All agent actions logged with correlation IDs
- [ ] Redaction policy applied to logs (no secrets/PII)
- [ ] Escalation path tested (critical finding triggers human alert)
- [ ] Architecture diagram (Mermaid) documents agent workflow

## Pitfalls to Watch
| Pitfall | Why It Matters | Detection |
|---------|---------------|-----------|
| Vague persona ("be helpful") | Agent scope is undefined | Review `.agent.md` for specificity |
| Allow-list without deny-list | Implicit access to new tools | Check for explicit deny section |
| No adversarial tests | Constraints not validated | Run injection test suite |
| Too many steps in plan | Quality degrades with depth | Count steps in plan (≤7) |
| Missing escalation path | Critical issues ignored | Test with known-critical input |
