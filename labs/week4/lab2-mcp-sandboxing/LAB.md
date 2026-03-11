# Lab 2: MCP Sandboxing & Security

## Objective
Enforce sandbox restrictions on MCP tools to prevent filesystem writes, path traversal, and secret leakage. Validate with a security test suite.

## Prerequisites
- Completed Lab 1 (MCP server with 3 tools exists)
- Understanding of filesystem permissions and path validation

## Tasks

### Task 1: Implement path validation

Add path safety checks to all tools:

```python
# path_validator.py
from pathlib import Path

class PathValidator:
    def __init__(self, workspace_root: Path):
        self.workspace_root = workspace_root.resolve()
    
    def validate(self, user_path: str) -> Path:
        """Ensure path is within workspace and doesn't escape via symlinks."""
        target = (self.workspace_root / user_path).resolve()
        
        # Check if resolved path is within workspace
        try:
            target.relative_to(self.workspace_root)
        except ValueError:
            raise PermissionError(
                f"Path '{user_path}' escapes workspace boundary"
            )
        
        return target
```

**Acceptance Criteria:**
- Path `../../etc/passwd` is rejected
- Path `/tmp/escape` is rejected
- Path `src/auth.py` (within workspace) is accepted

### Task 2: Make filesystem access read-only

Implement sandbox wrapper that prevents write operations:

```python
# sandbox.py
import tempfile
import shutil
from pathlib import Path

class ReadOnlySandbox:
    """Execute tools with read-only filesystem view."""
    
    def __init__(self, workspace_root: Path):
        self.workspace_root = workspace_root
    
    async def execute(self, tool_fn, args: dict) -> dict:
        """Run tool in read-only copy of workspace."""
        with tempfile.TemporaryDirectory() as tmpdir:
            # Copy workspace to temp location
            tmp_workspace = Path(tmpdir) / "workspace"
            shutil.copytree(self.workspace_root, tmp_workspace)
            
            # Make everything read-only
            for item in tmp_workspace.rglob("*"):
                if item.is_file():
                    item.chmod(0o444)  # r--r--r--
                else:
                    item.chmod(0o555)  # r-xr-xr-x
            
            # Execute tool with temp workspace
            modified_args = {**args, "workspace_root": tmp_workspace}
            return await tool_fn(modified_args)
```

**Acceptance:**
- Tool attempts to write a file → `PermissionError`
- Tool attempts to delete a file → `PermissionError`
- Tool can read all files normally

### Task 3: Implement response redaction

Add redaction filter to strip secrets before returning results:

```python
# redaction.py
import re
import json

class ResponseRedactor:
    """Remove secrets from tool responses."""
    
    SECRET_PATTERNS = [
        (r'ghp_[A-Za-z0-9]{36}', '[REDACTED_GITHUB_TOKEN]'),
        (r'sk-[A-Za-z0-9]{48}', '[REDACTED_API_KEY]'),
        (r'(?i)(password|secret|token|key)\s*[=:]\s*["\']([^"\']{8,})["\']', r'\1="[REDACTED]"'),
        (r'[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}', '[REDACTED_EMAIL]'),
    ]
    
    def redact(self, data: dict) -> dict:
        """Apply redaction patterns to response data."""
        json_str = json.dumps(data)
        
        for pattern, replacement in self.SECRET_PATTERNS:
            json_str = re.sub(pattern, replacement, json_str)
        
        return json.loads(json_str)
```

**Acceptance:**
- Finding contains `ghp_abc123...` → redacted to `[REDACTED_GITHUB_TOKEN]`
- Finding contains `password="secret123"` → redacted to `password="[REDACTED]"`
- Non-secret content remains unchanged

### Task 4: Build security test suite

Create `tests/test_security.py`:

```python
import pytest
from pathlib import Path
from mcp_server import scan_secrets, PathValidator, ReadOnlySandbox

class TestSandboxSecurity:
    """Adversarial tests for sandbox escape attempts."""
    
    def test_path_traversal_blocked(self):
        """Attempt to escape workspace via ../ path."""
        validator = PathValidator(Path("/workspace"))
        
        with pytest.raises(PermissionError):
            validator.validate("../../etc/passwd")
    
    def test_absolute_path_blocked(self):
        """Attempt to access absolute path outside workspace."""
        validator = PathValidator(Path("/workspace"))
        
        with pytest.raises(PermissionError):
            validator.validate("/tmp/secrets")
    
    def test_symlink_escape_blocked(self):
        """Symlink to outside workspace should be rejected."""
        # Setup: create symlink pointing outside workspace
        # Test: validator.validate() should raise PermissionError
        pass
    
    def test_file_write_blocked(self):
        """Tool attempting to write file should fail."""
        # Execute tool that tries to write
        # Assert PermissionError raised
        pass
    
    def test_file_delete_blocked(self):
        """Tool attempting to delete file should fail."""
        pass

class TestRedaction:
    """Validate secret redaction in responses."""
    
    def test_github_token_redacted(self):
        redactor = ResponseRedactor()
        response = {"findings": [{"token": "ghp_" + "A"*36}]}
        
        redacted = redactor.redact(response)
        assert "[REDACTED_GITHUB_TOKEN]" in str(redacted)
        assert "ghp_" not in str(redacted)
    
    def test_password_redacted(self):
        redactor = ResponseRedactor()
        response = {"config": {"password": "supersecret123"}}
        
        redacted = redactor.redact(response)
        assert "[REDACTED]" in str(redacted)
        assert "supersecret123" not in str(redacted)
    
    def test_clean_content_unchanged(self):
        redactor = ResponseRedactor()
        response = {"message": "No issues found"}
        
        redacted = redactor.redact(response)
        assert redacted == response
```

**Acceptance:** All security tests pass. No sandbox escapes possible.

### Task 5: Integrate sandbox into MCP server

Update `mcp_server.py` to use sandbox and redaction:

```python
# In call_tool handler
sandbox = ReadOnlySandbox(workspace_root=Path.cwd())
redactor = ResponseRedactor()

try:
    result = await sandbox.execute(tool_function, arguments)
    redacted_result = redactor.redact(result)
    return [TextContent(type="text", text=json.dumps(redacted_result))]
except Exception as e:
    # Return error with safe message
    return [TextContent(type="text", text=json.dumps({
        "error": str(e),
        "retryable": False
    }))]
```

**Acceptance:** All tool calls go through sandbox. All responses redacted.

## Evidence to Commit
- [ ] `mcp-server/path_validator.py`
- [ ] `mcp-server/sandbox.py`
- [ ] `mcp-server/redaction.py`
- [ ] `tests/test_security.py` — Security test suite
- [ ] `evidence/lab2-security-tests-pass.txt` — Test results
- [ ] `evidence/lab2-redaction-examples.json` — Before/after redaction

## Verification
```bash
# Run security tests
pytest tests/test_security.py -v

# Verify all tests pass
pytest tests/test_security.py && echo "SECURITY: PASS" || echo "SECURITY: FAIL"
```
