# Week 4: MCP Server — Review Checklist

## Per-Lab Review

### Lab 1: MCP Tools
- [ ] MCP server exposes tool discovery (JSON Schema available)
- [ ] All 3 tools have complete input schemas
- [ ] Tools are callable from Copilot/agent
- [ ] Output is structured JSON (not plain text)
- [ ] Tools validate inputs against declared schema

### Lab 2: Sandboxing & Security
- [ ] Filesystem access is read-only (write attempts blocked)
- [ ] Path traversal (`../../`) is prevented
- [ ] Network access is disabled/blocked
- [ ] Secrets are redacted from tool responses
- [ ] Security test suite passes (all escape attempts fail)

### Lab 3: Streaming & Error Contracts
- [ ] Timeout is enforced (30s max)
- [ ] Partial results returned when timeout hit
- [ ] All error codes tested (INVALID_INPUT, TIMEOUT, etc.)
- [ ] Streaming tool emits progress updates
- [ ] Correlation IDs present in all logs

## Cross-Cutting Checks
- [ ] All tool invocations logged with correlation ID
- [ ] Redaction policy applied before response sent
- [ ] No secrets or PII in server logs
- [ ] Architecture diagram (Mermaid) shows sandbox boundaries
- [ ] MCP server code committed with tests

## Pitfalls to Watch
| Pitfall | Why It Matters | Detection |
|---------|---------------|-----------|
| No timeout enforcement | Runaway tools block server | Test with infinite loop tool |
| Secrets in logs | Audit trail leaks credentials | Grep logs for patterns |
| Missing input validation | Agent sends malformed data, tool crashes | Schema validation test |
| Sandbox too permissive | Tools can write/delete files | Attempt file write in test |
| No error contracts | Agents don't know if retry is safe | Check for `retryable` field |
