# Resources

High-signal reference materials for advanced GitHub Copilot training.

---

## Week 1: Copilot + CLI for Automation

### Official Documentation
- [GitHub Copilot CLI Documentation](https://docs.github.com/en/copilot/github-copilot-in-the-cli)
- [Using GitHub Copilot in the CLI](https://docs.github.com/en/copilot/github-copilot-in-the-cli/using-github-copilot-in-the-cli)
- [GitHub CLI Manual](https://cli.github.com/manual/)

### Best Practices
- **Prompt templates**: Store reusable prompts in version control
- **Audit logging**: Capture all CLI sessions with timestamps
- **Rollback strategy**: Create Git tags before batch operations

---

## Week 2: Custom Agents

### Official Documentation
- [VS Code Agent Mode](https://code.visualstudio.com/docs/copilot/chat/chat-agent-mode)
- [Copilot Custom Instructions](https://docs.github.com/en/copilot/customizing-copilot/adding-repository-custom-instructions-for-github-copilot)
- [Agent Customization Files](https://code.visualstudio.com/docs/copilot/chat/chat-agent-mode#_agent-customization)

### Security Frameworks
- **STRIDE Threat Modeling**: [Microsoft STRIDE](https://docs.microsoft.com/en-us/azure/security/develop/threat-modeling-tool-threats)
- **OWASP Top 10**: [Current list](https://owasp.org/www-project-top-ten/)
- **RBAC Best Practices**: Least-privilege principle, denial-by-default

---

## Week 3: Skills

### Official Documentation
- [Copilot Custom Skills (SKILL.md)](https://code.visualstudio.com/docs/copilot/chat/chat-agent-mode)
- [Reusable Prompt Patterns](https://github.com/features/copilot/patterns)

### Dependency Management
- **Semantic Versioning**: [semver.org](https://semver.org/)
- **DAG Validation**: Detect cycles in skill dependencies
- **Backward Compatibility**: Migration guides for breaking changes

---

## Week 4: MCP Server

### Official Documentation
- [Model Context Protocol Specification](https://modelcontextprotocol.io/)
- [MCP Python SDK](https://modelcontextprotocol.io/docs/sdk/python)
- [Building MCP Servers](https://modelcontextprotocol.io/docs/concepts/servers)
- [MCP Security Best Practices](https://modelcontextprotocol.io/docs/concepts/transports)

### Implementation Patterns
- **Sandboxing**: Read-only filesystem, no network egress
- **Error Contracts**: Explicit error codes with retry guidance
- **Streaming**: Progress updates for long-running operations
- **Timeout Enforcement**: Kill runaway tools after threshold

---

## Cross-Cutting Topics

### Security
- [OWASP Secure Coding Practices](https://owasp.org/www-project-secure-coding-practices-quick-reference-guide/)
- [Secret Detection Tools](https://github.com/Yelp/detect-secrets)
- [CVSS v3.1 Calculator](https://www.first.org/cvss/calculator/3.1)

### Architecture
- [Clean Architecture (Robert C. Martin)](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [Domain-Driven Design Patterns](https://martinfowler.com/bliki/DomainDrivenDesign.html)
- [Hexagonal Architecture](https://alistair.cockburn.us/hexagonal-architecture/)

### Observability
- [Structured Logging Best Practices](https://www.loggly.com/ultimate-guide/python-logging-basics/)
- [Correlation IDs](https://hilton.org.uk/blog/microservices-correlation-id)
- [Redaction Policies](https://www.elastic.co/guide/en/elasticsearch/reference/current/redaction.html)

### Testing
- [Property-Based Testing (Hypothesis)](https://hypothesis.readthedocs.io/)
- [Mutation Testing (mutmut)](https://mutmut.readthedocs.io/)
- [Adversarial Testing for AI](https://github.com/openai/evals)

---

## Tools

### CLI & Automation
- **GitHub CLI**: `gh` command-line tool
- **jq**: JSON processor for CLI pipelines
- **detect-secrets**: Pre-commit hook for secret scanning

### Python Development
- **mypy**: Static type checker
- **pytest**: Testing framework
- **black**: Code formatter
- **ruff**: Fast Python linter

### Diagramming
- **Mermaid**: Markdown-based diagrams
- **PlantUML**: UML diagrams as code
- **draw.io**: Visual diagramming (desktop app)

### MCP Development
- **MCP Python SDK**: `pip install mcp`
- **MCP Inspector**: Debug MCP servers
- **JSON Schema Validator**: `pip install jsonschema`

---

## Reading List (High-Signal Only)

### Books
1. **"Clean Architecture"** by Robert C. Martin — DDD, layering, dependency rules
2. **"Threat Modeling: Designing for Security"** by Adam Shostack — STRIDE, attack trees
3. **"Building Secure and Reliable Systems"** by Google SRE — Production security patterns

### Papers
1. **"On the Dangers of Stochastic Parrots"** — AI limitations and risks
2. **"SoK: Security and Privacy in Machine Learning"** — Adversarial ML attacks
3. **"Prompt Injection Attacks"** — Attack taxonomy and mitigations

### Articles
1. [Prompt Engineering Guide](https://www.promptingguide.ai/) — Techniques and patterns
2. [AI Safety Guardrails](https://www.anthropic.com/index/constitutional-ai-harmlessness-from-ai-feedback) — Anthropic's approach
3. [Zero Trust Architecture](https://www.nist.gov/publications/zero-trust-architecture) — NIST SP 800-207

---

## Community & Support

### GitHub Discussions
- [GitHub Copilot Discussions](https://github.com/orgs/community/discussions/categories/copilot)
- [MCP Community](https://github.com/modelcontextprotocol/discussions)

### Discord/Slack
- GitHub Copilot Discord (enterprise)
- MCP Developers Slack

### Stack Overflow
- Tag: `github-copilot`
- Tag: `model-context-protocol`

---

## Templates & Examples

### PR Template with AI-Assisted Changes
```markdown
## Description
<!-- Brief description of changes -->

## AI-Assisted Changes
- [ ] This PR includes AI-generated code
- Model used: GitHub Copilot (Claude Sonnet 4.5)
- Prompt hash: `abc123...`
- Human review completed: Yes/No

## Architecture Conformance
- [ ] Respects clean architecture layers
- [ ] No new dependencies violating DDD boundaries
- [ ] All changes covered by tests

## Security Review
- [ ] No secrets in code or commits
- [ ] Input validation added where needed
- [ ] Threat model updated (if applicable)

## Reviewer Checklist
- [ ] AI-generated code manually reviewed line-by-line
- [ ] Tests cover edge cases and adversarial inputs
- [ ] Performance impact assessed
```

### Architecture Decision Record (ADR) Template
```markdown
# ADR-001: [Title]

## Status
Proposed | Accepted | Deprecated | Superseded

## Context
<!-- What is the issue we're addressing? -->

## Decision
<!-- What is the change we're proposing? -->

## Consequences
### Positive
- 

### Negative
- 

### Risks
- 

## Alternatives Considered
1. 
2. 

## Compliance
- [ ] RBAC constraints respected
- [ ] Audit logging in place
- [ ] Rollback plan documented
```

### Skill Template
```markdown
---
name: skill-name
description: One-line description
version: 1.0.0
parameters:
  - name: param1
    type: string
    required: true
    description: What this parameter does
dependencies: []
tags: []
---

# Skill: [Name]

## Purpose
<!-- Why this skill exists -->

## Procedure
1. 
2. 
3. 

## Output Schema
\`\`\`json
{
  "skill": "skill-name",
  "result": {}
}
\`\`\`

## Failure Modes
- 

## Test Cases
1. 
```

---

## Glossary

| Term | Definition |
|------|------------|
| **RBAC** | Role-Based Access Control — permissions based on user roles |
| **STRIDE** | Spoofing, Tampering, Repudiation, Info Disclosure, DoS, Elevation of Privilege |
| **MCP** | Model Context Protocol — standardized interface for AI tool connectivity |
| **DAG** | Directed Acyclic Graph — dependency structure without cycles |
| **Semver** | Semantic Versioning — MAJOR.MINOR.PATCH version scheme |
| **PII** | Personally Identifiable Information — data requiring protection |
| **DDD** | Domain-Driven Design — software design approach focusing on domain model |
| **CVSS** | Common Vulnerability Scoring System — standard for vulnerability severity |
| **Idempotent** | Operation that produces same result when repeated |
| **Correlation ID** | Unique identifier tracking request across system components |
