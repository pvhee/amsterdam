# Cutover MCP Improvement Plan — Based on PagerDuty MCP Comparison

## Context

Comparing [gocutover/cutover-mcp-public](https://github.com/gocutover/cutover-mcp-public) (Python, FastMCP, v0.2.0) against [PagerDuty/pagerduty-mcp-server](https://github.com/PagerDuty/pagerduty-mcp-server) (Python, FastMCP, v0.15.1) — both in the incident/ops management domain.

---

## 1. Detailed Comparison

### Architecture

| Aspect | Cutover MCP | PagerDuty MCP | Gap |
|---|---|---|---|
| Framework | FastMCP | FastMCP | Same |
| Python | 3.13+ | 3.12 | Minor |
| Tool count | ~10 modules (~20 tools) | 14 modules (60+ tools) | PD has broader API coverage |
| MCP Resources | Stubbed (`resources/__init__.py` only) | Not used | Neither leverages MCP Resources |
| MCP Prompts | None | None | Neither uses MCP Prompts |
| Transport | STDIO (Docker hints at HTTP) | STDIO | Same |
| API client | Custom httpx wrapper | Official `pagerduty` SDK (v5.3) | PD leverages official SDK |
| Data models | JSON:API Pydantic generics | Pydantic models with `.to_params()` | Cutover's is more rigorous |
| Schema for LLM | `inject_return_schema` decorator | Standard type hints | Cutover has advantage here |

### Safety & Permissions

| Aspect | Cutover MCP | PagerDuty MCP | Gap |
|---|---|---|---|
| Read/write separation | None — all tools always enabled | `--enable-write-tools` flag; writes disabled by default | **Critical gap** |
| Tool annotations | None | `readOnlyHint`, `destructiveHint`, `idempotentHint` on every tool | **Critical gap** |
| Server instructions | Generic ("A set of tools and resources...") | Detailed guidance ("first get user data, scope by user id") | **Important gap** |
| Destructive action warnings | None | Annotations let MCP clients show confirmation dialogs | **Critical gap** |

### Developer Experience

| Aspect | Cutover MCP | PagerDuty MCP | Gap |
|---|---|---|---|
| CLI entry point | `uv run python src/cutover_mcp/server.py` | `pagerduty-mcp` script via pyproject.toml | **Moderate gap** |
| Evals / quality testing | pytest unit tests | pytest + `pagerduty_mcp_evals/` (MCP-specific eval suite) | **Important gap** |
| Multi-region | `CUTOVER_BASE_URL` env var | Explicit US/EU endpoint docs | Similar |
| Docker | Yes (port 8000) | Yes | Same |
| IDE config examples | Claude Desktop + VS Code | Claude Desktop + VS Code + Cursor + Bedrock | PD covers more clients |
| Context/multi-tenant | Singleton `APIClientManager` | Strategy pattern (Application vs Request context) | **Moderate gap** |

### What Cutover Does Better

- **JSON:API Pydantic generics** — proper `JsonApiObject[T]`, `JsonApiListResponse[T]` hierarchy with relationship modeling. PagerDuty's models are flatter.
- **`inject_return_schema` decorator** — generates compact schema text and injects into tool docstrings, helping LLMs parse responses with fewer tokens. PagerDuty doesn't do this.
- **Retry with exponential backoff** — built into API client with smart 4xx handling (fail-fast on client errors, retry on 429/transport errors). PagerDuty delegates to the SDK.
- **Lifecycle management** — proper FastMCP lifespan context manager for client pool init/cleanup.

---

## 2. Recommended Improvements (Priority Order)

### P0 — Critical (Safety & Enterprise Readiness)

#### 2.1 Add read/write tool separation
- Add `--enable-write-tools` CLI flag (default: disabled)
- Classify each tool as read-only or write
- Only register write tools when flag is passed
- **Why**: Enterprise customers need safe defaults. An LLM accidentally calling `manage_runbook(action="cancel")` could disrupt production operations.
- **Reference**: PagerDuty's `server.py` — `add_read_only_tool()` vs `add_write_tool()` pattern

#### 2.2 Add MCP tool annotations
- Annotate every tool with `readOnlyHint`, `destructiveHint`, `idempotentHint`
- Read tools: `readOnlyHint=True, destructiveHint=False, idempotentHint=True`
- Write tools: `readOnlyHint=False, destructiveHint=True, idempotentHint=False`
- Idempotent writes (like update): `destructiveHint=False, idempotentHint=True`
- **Why**: MCP clients use these annotations to show confirmation dialogs and prevent accidental mutations.

#### 2.3 Improve server instructions
- Replace generic description with operational guidance
- Include: "Always identify the workspace first. List available runbooks before taking action. Confirm with the user before starting, pausing, or cancelling runbooks."
- **Why**: Good instructions dramatically improve LLM tool selection and reduce errors.

### P1 — Important (Quality & DX)

#### 2.4 Add proper CLI entry point
- Add `[project.scripts]` to `pyproject.toml`: `cutover-mcp = "cutover_mcp.server:main"`
- Create a `main()` function with argument parsing (e.g., `--enable-write-tools`)
- **Why**: `cutover-mcp` is cleaner than `uv run python src/cutover_mcp/server.py`

#### 2.5 Add MCP eval suite
- Create `cutover_mcp_evals/` directory
- Write scenario-based evals: "List runbooks in workspace X", "Create a runbook from template Y", "Start a runbook and monitor progress"
- Test tool selection accuracy, parameter extraction, and response parsing
- **Why**: Unit tests verify code works; evals verify the LLM experience works.

#### 2.6 Add context strategy pattern for multi-tenant support
- Extract client creation into a strategy interface
- Support `ApplicationContextStrategy` (current singleton, env-var based) and `RequestContextStrategy` (per-request credentials)
- **Why**: Enables embedding Cutover MCP in shared services or multi-tenant agent platforms.

### P2 — Nice to Have (Completeness)

#### 2.7 Expand IDE config examples
- Add Cursor config example
- Add AWS Bedrock config example
- Document HTTP/SSE transport setup (since Dockerfile already exposes port 8000)
- **Why**: Lower barrier to adoption.

#### 2.8 Implement MCP Resources
- The `resources/` directory is currently a stub
- Consider exposing read-only data as MCP Resources (e.g., workspace list, runbook templates)
- Resources are better than tools for static/slowly-changing data — they can be cached
- **Why**: MCP Resources are underutilized across the ecosystem; this would be differentiating.

#### 2.9 Add MCP Prompts
- Neither Cutover nor PagerDuty uses MCP Prompts yet
- Consider adding prompt templates for common workflows: "Incident response playbook", "DR failover checklist", "Post-incident review"
- **Why**: Prompts guide users toward best-practice workflows. Differentiating feature.

---

## 3. Implementation Order

```
Phase 1 (Safety):     2.1 read/write separation → 2.2 tool annotations → 2.3 server instructions
Phase 2 (DX):         2.4 CLI entry point → 2.5 eval suite
Phase 3 (Scale):      2.6 multi-tenant context strategy
Phase 4 (Differentiation): 2.7 IDE configs → 2.8 MCP Resources → 2.9 MCP Prompts
```

---

## 4. Summary

Cutover MCP has strong fundamentals (JSON:API models, schema injection, retry logic) but lacks the safety guardrails and developer experience polish that PagerDuty's more mature server provides. The highest-impact improvements are **read/write separation** and **tool annotations** — these are table-stakes for enterprise adoption and can be implemented quickly without architectural changes.
