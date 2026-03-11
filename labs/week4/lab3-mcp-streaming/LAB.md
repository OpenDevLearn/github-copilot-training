# Lab 3: MCP Streaming & Error Contracts

## Objective
Implement streaming tool responses for long-running operations, define explicit error contracts, and enforce timeout handling with partial results.

## Prerequisites
- Completed Labs 1 and 2 (sandboxed MCP server with 3 tools)
- Understanding of async/await and generators in Python

## Tasks

### Task 1: Define error contract enum

Create `error_contracts.py`:

```python
# error_contracts.py
from enum import Enum
from dataclasses import dataclass

class ToolErrorCode(Enum):
    """Standardized error codes for all tools."""
    INVALID_INPUT = "INVALID_INPUT"
    PATH_NOT_FOUND = "PATH_NOT_FOUND"
    PERMISSION_DENIED = "PERMISSION_DENIED"
    TIMEOUT = "TIMEOUT"
    RESOURCE_EXHAUSTED = "RESOURCE_EXHAUSTED"
    INTERNAL_ERROR = "INTERNAL_ERROR"

@dataclass
class ToolError:
    """Structured error response."""
    code: ToolErrorCode
    message: str
    retryable: bool
    details: dict = None
    
    def to_json(self) -> dict:
        return {
            "error": {
                "code": self.code.value,
                "message": self.message,
                "retryable": self.retryable,
                "details": self.details or {}
            }
        }

def wrap_error(e: Exception, context: str = "") -> ToolError:
    """Convert exception to ToolError."""
    if isinstance(e, FileNotFoundError):
        return ToolError(
            ToolErrorCode.PATH_NOT_FOUND,
            f"Path not found: {context}",
            retryable=False
        )
    elif isinstance(e, PermissionError):
        return ToolError(
            ToolErrorCode.PERMISSION_DENIED,
            f"Permission denied: {context}",
            retryable=False
        )
    elif isinstance(e, TimeoutError):
        return ToolError(
            ToolErrorCode.TIMEOUT,
            f"Operation timed out: {context}",
            retryable=True
        )
    else:
        return ToolError(
            ToolErrorCode.INTERNAL_ERROR,
            f"Unexpected error: {str(e)}",
            retryable=True
        )
```

**Acceptance:**
- All error codes defined
- Each error has `retryable` boolean
- Errors are JSON-serializable

### Task 2: Implement timeout enforcement

Add timeout wrapper for all tool executions:

```python
# timeout_handler.py
import asyncio
import functools

def with_timeout(timeout_seconds: int = 30):
    """Decorator to enforce timeout on tool execution."""
    def decorator(func):
        @functools.wraps(func)
        async def wrapper(*args, **kwargs):
            try:
                return await asyncio.wait_for(
                    func(*args, **kwargs),
                    timeout=timeout_seconds
                )
            except asyncio.TimeoutError:
                raise ToolError(
                    ToolErrorCode.TIMEOUT,
                    f"Execution exceeded {timeout_seconds}s timeout",
                    retryable=True
                )
        return wrapper
    return decorator

# Usage
@with_timeout(timeout_seconds=30)
async def scan_secrets(args: dict):
    # Tool implementation
    pass
```

**Acceptance:**
- Tool exceeding 30s is killed
- Timeout error has `retryable: true`

### Task 3: Implement streaming for large scans

Convert `scan_secrets` to stream progress updates:

```python
# streaming_scan.py
async def scan_secrets_streaming(args: dict):
    """Stream findings as files are scanned (not all at once)."""
    target = Path(args["target_path"])
    files = list(target.rglob("*.py"))
    total = len(files)
    
    findings = []
    
    for i, file in enumerate(files, 1):
        # Scan file
        file_findings = await scan_file(file)
        findings.extend(file_findings)
        
        # Stream progress every 10 files or at the end
        if i % 10 == 0 or i == total:
            yield {
                "type": "progress",
                "processed": i,
                "total": total,
                "findings_count": len(findings),
                "latest_file": str(file)
            }
    
    # Final result
    yield {
        "type": "final",
        "summary": {
            "total_files": total,
            "total_findings": len(findings)
        },
        "findings": findings
    }

# In MCP server call_tool handler
async def call_tool_streaming(name: str, args: dict):
    if name == "scan_secrets":
        async for chunk in scan_secrets_streaming(args):
            yield TextContent(type="text", text=json.dumps(chunk))
    # ... other tools
```

**Acceptance:**
- Progress updates emitted during scan (not just at end)
- Final result includes all findings
- Agent sees incremental progress

### Task 4: Test all error codes

Create `tests/test_error_contracts.py`:

```python
import pytest
from mcp_server import call_tool
from error_contracts import ToolErrorCode

@pytest.mark.asyncio
async def test_invalid_input_error():
    """Missing required parameter → INVALID_INPUT."""
    result = await call_tool("scan_secrets", {})  # missing target_path
    
    assert "error" in result
    assert result["error"]["code"] == ToolErrorCode.INVALID_INPUT.value
    assert result["error"]["retryable"] is False

@pytest.mark.asyncio
async def test_path_not_found_error():
    """Nonexistent path → PATH_NOT_FOUND."""
    result = await call_tool("scan_secrets", {"target_path": "nonexistent/"})
    
    assert result["error"]["code"] == ToolErrorCode.PATH_NOT_FOUND.value
    assert result["error"]["retryable"] is False

@pytest.mark.asyncio
async def test_permission_denied_error():
    """Path outside workspace → PERMISSION_DENIED."""
    result = await call_tool("scan_secrets", {"target_path": "../../etc"})
    
    assert result["error"]["code"] == ToolErrorCode.PERMISSION_DENIED.value
    assert result["error"]["retryable"] is False

@pytest.mark.asyncio
async def test_timeout_error():
    """Long-running tool → TIMEOUT."""
    # Create a tool that sleeps 40s (exceeds 30s timeout)
    result = await call_tool("slow_tool", {"duration": 40})
    
    assert result["error"]["code"] == ToolErrorCode.TIMEOUT.value
    assert result["error"]["retryable"] is True
```

**Acceptance:** All error codes reachable. Test suite passes.

### Task 5: Add correlation IDs to all logs

Implement correlation ID tracking:

```python
# correlation.py
import uuid
import contextvars

correlation_id = contextvars.ContextVar("correlation_id", default=None)

def generate_correlation_id() -> str:
    """Generate new correlation ID."""
    return f"mcp-{uuid.uuid4()}"

def log_tool_invocation(tool_name: str, args: dict, result: dict, duration: float):
    """Log with correlation ID."""
    cid = correlation_id.get()
    print(f"[{cid}] tool={tool_name} duration={duration:.2f}s args_hash={hash_args(args)}")
    print(f"[{cid}] result_summary={summarize_result(result)}")

# In MCP server
async def call_tool(name: str, args: dict):
    cid = generate_correlation_id()
    correlation_id.set(cid)
    
    start = time.time()
    try:
        result = await execute_tool(name, args)
        duration = time.time() - start
        log_tool_invocation(name, args, result, duration)
        return result
    except Exception as e:
        duration = time.time() - start
        log_error(name, args, e, duration)
        raise
```

**Acceptance:**
- Every tool call has unique correlation ID
- All logs include correlation ID prefix
- Logs are grep-able by correlation ID

## Evidence to Commit
- [ ] `mcp-server/error_contracts.py`
- [ ] `mcp-server/timeout_handler.py`
- [ ] `mcp-server/streaming_scan.py`
- [ ] `mcp-server/correlation.py`
- [ ] `tests/test_error_contracts.py` — Error code tests
- [ ] `evidence/lab3-streaming-output.jsonl` — Streaming progress example
- [ ] `evidence/lab3-timeout-test.json` — Timeout error example
- [ ] `evidence/lab3-correlation-logs.txt` — Logs with correlation IDs

## Verification
```bash
# Run error contract tests
pytest tests/test_error_contracts.py -v

# Verify all error codes tested
pytest tests/test_error_contracts.py --collect-only | grep -c "test_.*_error" | grep -q "4" && echo "ERROR TESTS: PASS" || echo "ERROR TESTS: FAIL"

# Verify correlation IDs in logs
grep -E "^\[mcp-[a-f0-9-]+\]" evidence/lab3-correlation-logs.txt && echo "CORRELATION: PASS" || echo "CORRELATION: FAIL"
```
