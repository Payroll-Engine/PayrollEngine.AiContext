# Payroll Engine AI Context

Structured LLM context for **MCP Server access** and **REST API integration** with Payroll Engine.

> **License:** MIT — free for personal and commercial use.

## What is this?

Pre-built context files optimised for LLMs (Claude, GPT-4, Copilot, etc.) when
working with the Payroll Engine MCP Server or REST API. Instead of explaining PE
concepts from scratch in every session, load the relevant context as system prompt
or project knowledge.

For **regulation development** (Exchange JSON/YAML, No-Code, C# scripting), see
[PayrollEngine.AiContext.Pro](https://payrollengine.org) — included in MCP Server Pro.

## Context × Role Matrix

| Context file | Description | Provider | Automator |
|:-------------|:------------|:--------:|:---------:|
| `01-pe-overview.md` | Core concepts, architecture, MCP access control | ✓ | ✓ |
| `05-mcp-tools.md` | MCP Server read-only tool inventory | ✓ | ✓ |
| `prompts/provider.md` | System prompt for REST API integration | ✓ | |
| `prompts/automator.md` | System prompt for CLI / Client SDK automation | | ✓ |

**Role definitions:**
- **Provider** — EOR / Bureau / HCM vendor integrating via REST API or MCP Server
- **Automator** — DevOps / integration engineer using `pecmd` CLI and Client SDK

## Repository structure

```
context/
  01-pe-overview.md     Core concepts, architecture, MCP access control
  05-mcp-tools.md       MCP Server read-only tool inventory  [generated]
prompts/
  provider.md           System prompt — REST API integration
  automator.md          System prompt — CLI / Client SDK automation
devops/
  Generate-AiContext.ps1   Regenerate [generated] files from source repos
```

Files marked `[generated]` are overwritten by `Generate-AiContext.ps1` on each release.

## Usage

### Claude.ai Project (recommended)

Upload the relevant context files as Project Knowledge, then use the matching prompt:

| Goal | Context files | Prompt |
|:-----|:-------------|:-------|
| MCP Server queries | `01-pe-overview.md`, `05-mcp-tools.md` | `prompts/provider.md` |
| CLI / SDK automation | `01-pe-overview.md` | `prompts/automator.md` |

### API system prompt

```csharp
var systemPrompt = await File.ReadAllTextAsync("prompts/provider.md")
                 + await File.ReadAllTextAsync("context/01-pe-overview.md")
                 + await File.ReadAllTextAsync("context/05-mcp-tools.md");
```

## Regenerating generated files

```powershell
.\devops\Generate-AiContext.ps1
```
