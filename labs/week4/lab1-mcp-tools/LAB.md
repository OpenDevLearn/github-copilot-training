# Lab 1: MCP Tools — Author Custom Tools

## Objective
Build an MCP server in Python with 3 custom security tools, expose them via JSON Schema, and invoke them from Copilot.

## Prerequisites
- Python 3.10+
- `mcp` Python SDK installed (`pip install mcp`)
- Understanding of JSON Schema

## Tasks

### Task 1: Set up MCP server scaffold

```bash
mkdir -p labs/week4/lab1-mcp-tools/mcp-server
cd labs/week4/lab1-mcp-tools/mcp-server

# Create virtual environment
python -m venv .venv
source .venv/bin/activate  # or .venv\Scripts\activate on Windows

# Install MCP SDK
pip install mcp
pip freeze > requirements.txt
```

**Acceptance:** Virtual environment activated, MCP SDK installed.

### Task 2: Implement Tool 1 — `scan_secrets`

Create `mcp_server.py` and implement:
- **Input**: `target_path` (string, required), `patterns` (array of strings, optional)
- **Output**: JSON with `findings` array (file, line, pattern, severity)
- **Behavior**: Scan Python files for hardcoded secrets using regex patterns

```python
# Implement scan_secrets tool
# See module README Pattern 1 for reference implementation
```

**Acceptance:**
- Tool registered in `list_tools()`
- Input schema is valid JSON Schema
- Returns structured JSON output

### Task 3: Implement Tool 2 — `threat_model`

- **Input**: `component` (string), `entry_points` (array of strings)
- **Output**: JSON with STRIDE threat model
- **Behavior**: Generate threat questions for each STRIDE category

**Acceptance:**
- Tool produces 6 threats (one per STRIDE category)
- Output includes mitigation guidance

### Task 4: Implement Tool 3 — `analyze_architecture`

- **Input**: `target_path` (string), `architecture_config` (string, optional)
- **Output**: JSON with layer violations
- **Behavior**: Check if imports respect clean architecture rules

**Acceptance:**
- Reuses logic from Week 3 Skills lab
- Detects at least one known violation in sample codebase

### Task 5: Test tool discovery

Run the MCP server and verify tools are discoverable:

```bash
# Start server
python mcp_server.py

# In another terminal, test discovery (manual JSON-RPC call or via Copilot)
```

**Acceptance:**
- `list_tools()` returns 3 tools
- Each tool has valid JSON Schema
- Tool descriptions are clear and actionable

### Task 6: Invoke tools from Copilot

Configure VS Code to use the MCP server:

```json
// .vscode/settings.json
{
  "github.copilot.advanced": {
    "mcp": {
      "servers": {
        "security-tools": {
          "command": "python",
          "args": ["labs/week4/lab1-mcp-tools/mcp-server/mcp_server.py"]
        }
      }
    }
  }
}
```

Invoke via Copilot Chat:
```
@agent Use the scan_secrets tool on src/
@agent Generate a threat model for the authentication component with entry points: login, logout, refresh_token
```

**Acceptance:**
- Copilot discovers all 3 tools
- Tools execute and return structured results
- No errors in MCP server logs

## Evidence to Commit
- [ ] `mcp-server/mcp_server.py` — MCP server implementation
- [ ] `mcp-server/requirements.txt` — Python dependencies
- [ ] `evidence/lab1-tool-discovery.json` — Output of `list_tools()`
- [ ] `evidence/lab1-scan-secrets-result.json` — Sample invocation
- [ ] `evidence/lab1-threat-model-result.json` — Sample invocation
- [ ] `evidence/lab1-analyze-arch-result.json` — Sample invocation

## Verification
```bash
# Verify all tools return valid JSON
python -m json.tool evidence/lab1-scan-secrets-result.json > /dev/null && echo "PASS" || echo "FAIL"
python -m json.tool evidence/lab1-threat-model-result.json > /dev/null && echo "PASS" || echo "FAIL"
python -m json.tool evidence/lab1-analyze-arch-result.json > /dev/null && echo "PASS" || echo "FAIL"

# Verify tool count
grep -c '"name":' evidence/lab1-tool-discovery.json | grep -q "3" && echo "TOOL COUNT: PASS" || echo "TOOL COUNT: FAIL"
```
