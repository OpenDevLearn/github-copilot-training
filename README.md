# Advanced GitHub Copilot Training Curriculum

## Target Audience
Principal Software Architects (10–15 years experience)

## Focus Areas
This curriculum covers **4 advanced modules** across **4 weeks**:

| Week | Module | What It Is | What You Build |
|------|--------|-----------|----------------|
| 1 | **Copilot + CLI** | Use Copilot in the **terminal** (`gh copilot suggest/explain`) to automate shell commands, Git workflows, and batch file operations. You type natural language; Copilot generates and runs shell commands. | CLI scripts that refactor code across repos, create branches/PRs via natural language, and batch-process files with human approval gates. |
| 2 | **Custom Agents** | Define **AI personas** (`.agent.md` files) that constrain what Copilot can do inside VS Code — which tools it can use, which files it can access, and how many steps it takes before asking you. Think of it as **RBAC for the AI.** | A security-auditor agent that can only read code (no writes), follows a multi-step review plan, and stops at checkpoints for your approval. |
| 3 | **Skills** | Author **reusable instruction sets** (`SKILL.md` files) that agents can invoke by name — like functions for AI behavior. A skill takes typed parameters, follows a procedure, and returns structured output. Skills chain together. | A 3-skill pipeline: detect architecture violations → find injection vulnerabilities → suggest remediations. With semantic versioning and dependency graphs. |
| 4 | **MCP Server** | Build a **standalone Python server** that exposes custom tools to Copilot via the Model Context Protocol. The agent discovers and calls your tools over JSON-RPC. You control sandboxing, timeouts, and error handling. | A security-tools MCP server with 3 tools (`scan_secrets`, `threat_model`, `analyze_architecture`), sandbox enforcement, streaming responses, and correlation-ID logging. |

### How They Relate
```
You (terminal)          You (VS Code)
    │                       │
    ▼                       ▼
 Copilot CLI ───────► Custom Agent ───► Skills (reusable instructions)
 (Week 1)              (Week 2)         (Week 3)
 "run this              "AI persona        │
  shell command"         with rules"        │ invoke tools
                                            ▼
                                        MCP Server (Week 4)
                                        "sandboxed tool execution"
```
- **CLI** = you talk to Copilot in the terminal to run commands and scripts
- **Agents** = you configure an AI persona with constrained behavior inside VS Code
- **Skills** = reusable procedures that agents call — the "what to do" logic
- **MCP Server** = external tools that agents/skills call — the "how to execute" layer

### Scope & Storage

| Module | Global or Repo-Specific? | Where It Lives | Shared Across Repos? |
|--------|--------------------------|----------------|----------------------|
| **Copilot CLI** | **Global** — `gh copilot` is a system-wide CLI extension | Installed via `gh extension install github/gh-copilot`. No per-repo config needed. Prompt templates _can_ be stored in a repo (`prompts/`) for reuse. | Yes — the CLI works in any terminal, any directory. Not tied to a repo. |
| **Custom Agents** | **Both** — can be user-global or repo-scoped | **Repo-scoped:** `.github/copilot-instructions.md` (applies to everyone who opens this repo) or `.github/agents/*.md` (named agents per repo). **User-scoped:** VS Code user settings (`settings.json` → `github.copilot.chat.codeGeneration.instructions`). | Repo-scoped agents travel with the repo (anyone who clones gets them). User-scoped instructions apply to all repos you open. |
| **Skills** | **Repo-scoped** | `SKILL.md` files stored in the repo (e.g., `skills/enforce-clean-architecture/SKILL.md`). Discovered by Copilot when the repo is open in VS Code. | Not automatically shared. To reuse across repos, copy the `SKILL.md` files or publish to a shared repo that teams reference. |
| **MCP Server** | **Both** — server code is repo-scoped, but registration can be user-global | **Server code:** lives in a repo or standalone project (e.g., `mcp-server/server.py`). **Registration:** configured in `.vscode/settings.json` (workspace-scoped) or VS Code user settings (global). `.vscode/mcp.json` for workspace-level registration. | The server process itself can serve multiple workspaces if registered globally. The code is repo-scoped unless deployed as a shared service. |

**Key distinction:**
- **Instructions & agents** = text files that tell Copilot _how to behave_ → stored in `.github/` (repo) or VS Code settings (user)
- **Skills** = text files that define _reusable procedures_ → stored in the repo, discovered locally
- **MCP servers** = running processes that provide _executable tools_ → code in a repo, registered in VS Code settings

## Structure
```
├── curriculum/              # Main curriculum document with roadmap
├── modules/                 # Week-by-week module content
│   ├── week1-copilot-cli/
│   ├── week2-custom-agents/
│   ├── week3-skills/
│   └── week4-mcp-server/
├── labs/                    # Hands-on exercises (repo-first)
├── capstone/                # Final project
└── resources/               # Reference materials
```

## Getting Started
1. Read [curriculum/CURRICULUM.md](curriculum/CURRICULUM.md) for the complete roadmap
2. Follow weekly modules in sequence
3. Complete labs with binary acceptance criteria
4. Build capstone project applying all concepts

## Constraints
- Security-first, regulated environment
- Enterprise governance, auditable changes
- RBAC, zero-trust APIs
- GitHub Enterprise Cloud + Copilot (Chat + CLI)
- 5–7 hours/week commitment

## Evaluation
Outcomes-driven with binary acceptance criteria and rubrics. No beginner content.
