> ⚠️ **SUPERSEDED BY v19.1.5** — 2026-04-23. This file is v18.1-era content retained for historical reference. The authoritative v19.1.5 Bible at `https://github.com/ARJ999/MCP-God-Agent-Development-Bible` (branch `main`, flat layout) takes precedence on every rule below. Mapping:
> - Laws + architecture → `00_PHILOSOPHY.md` + `01_INVIOLABLE_LAWS.md` + `02_ARCHITECTURE.md`
> - Server/skill/tool standards → `03_MCP_SERVER_STANDARD.md` + `04_SKILL_STANDARD.md` + `05_TOOL_STANDARD.md`
> - Pairing Contract + 10 Guardrails (G2 removed v19.1.2) → `07_PAIRING_CONTRACT.md`
> - Upgrade Playbook (9 phases + Phase 1.5) → `08_UPGRADE_PLAYBOOK.md`
> - Completion Gate → `v19.0/09_VERIFICATION_PROTOCOL.md`
> - Upstream sync → `v19.0/13_UPSTREAM_SYNC_PROTOCOL.md`
>
> When these reference files contradict the parent `SKILL.md` or v19.0 Bible, the Bible wins.

---

# Canonical Standards Reference

> Authoritative reference for MCP server development. Three sections: Seven Unbreakable Laws, Gold Stack v18.1, 4-Tier Architecture v3.0.

---

## Section 1: Seven Unbreakable Laws

Every MCP server MUST comply with all seven laws. No exceptions, no shortcuts, no "we'll fix it later."

---

### Law 1: Singleton with `__new__` Method

**Rule**: API clients MUST use `__new__` with `_instance = None` and `_initialized = False` class variables.

**WHY**: Prevents resource leaks from multiple connection instances. A second client means a second connection pool, doubled memory, and race conditions on shared state.

**CORRECT**:
```python
class APIClient:
    _instance = None
    _initialized = False

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

    def __init__(self):
        if self._initialized:
            return
        self.__class__._initialized = True
        # initialization code here
```

**WRONG**:
```python
# Dict-based pattern — creates separate instances per key, defeats purpose
_instances = {}

# No singleton at all — every import creates a new client
client = APIClient()  # in multiple files
```

**Verify**:
```bash
grep -rn "_instance = None" src/ && grep -rn "__new__" src/
```

---

### Law 2: Lazy Connection Initialization

**Rule**: Connections MUST be established on first use, never at import time.

**WHY**: Prevents startup crashes when services are temporarily unavailable. The server boots clean and connects only when a tool actually needs data.

**CORRECT**:
```python
async def get_connection(self, org_alias: str = "default"):
    if org_alias not in self._connections:
        self._connections[org_alias] = await self._create_connection(org_alias)
    return self._connections[org_alias]
```

**WRONG**:
```python
# Connecting in __init__ — crashes if service is down at startup
def __init__(self):
    self.conn = create_connection()  # VIOLATION

# Module-level connection — same problem, worse timing
conn = create_connection()  # runs at import
```

**Verify**:
```bash
grep -rn "def __init__" src/api_client.py
# Should NOT contain connect/session creation calls
```

---

### Law 3: Async Executor Wrapping (`asyncio.to_thread`)

**Rule**: ALL synchronous SDK/API calls MUST use `asyncio.to_thread()`. Mandatory since v18.1.

**WHY**: Blocking calls freeze the event loop, killing all concurrent operations. One slow API call stalls every other tool invocation in flight.

**CORRECT**:
```python
# asyncio.to_thread — pass function and args separately
result = await asyncio.to_thread(conn.query, soql_query)
```

**WRONG**:
```python
# run_in_executor — verbose, error-prone, no automatic contextvars copy
result = await loop.run_in_executor(None, conn.query, soql_query)

# Direct sync call — blocks the entire event loop
result = conn.query(soql_query)  # VIOLATION
```

**Verify**:
```bash
grep -rn "run_in_executor" src/ && echo "VIOLATION" || echo "PASS"
```

---

### Law 4: NO Duplicate Singleton Declarations

**Rule**: `api_client = APIClient()` declared ONCE at module bottom, imported everywhere.

**WHY**: Multiple declarations create separate instances, defeating singleton. Even with `__new__`, scattered instantiation makes code fragile and confusing.

**CORRECT**:
```python
# api_client.py — bottom of file
api_client = APIClient()

# tools/accounts/operations.py
from mcp_server.core.api_client import api_client
```

**FastMCP 3.0 safety net**: `on_duplicate="error"` catches tool registration conflicts.

**Verify**:
```bash
grep -rn "APIClient()" src/ | wc -l
# Must be exactly 1
```

---

### Law 5: NO `**kwargs` in Tool Handlers

**Rule**: Every parameter MUST be explicitly typed with name and type annotation.

**WHY**: LLMs cannot discover parameters hidden in `**kwargs` — they become invisible. The model sees no parameter names, no types, no defaults. The tool is effectively broken for AI callers.

**CORRECT**:
```python
@mcp.tool()
async def get_account(account_id: str, include_contacts: bool = False) -> str:
    """Retrieve a Salesforce account by ID."""
    ...
```

**WRONG**:
```python
@mcp.tool()
async def get_account(**kwargs) -> str:
    """Retrieve a Salesforce account by ID."""
    account_id = kwargs.get("account_id")  # INVISIBLE to LLM
    ...
```

**Verify**:
```bash
grep -rn "\*\*kwargs" src/tools/ && echo "VIOLATION" || echo "PASS"
```

---

### Law 6: MANDATORY Stateless HTTP Mode

**Rule**: `stateless_http=True` in `mcp.run()` AND `transport="streamable-http"`.

**WHY**: Stateful sessions cause affinity problems, memory leaks, and scaling failures. Stateless HTTP lets any replica handle any request — horizontal scaling just works.

**CORRECT**:
```python
mcp = FastMCP("my-server", version="1.0.0", on_duplicate="error")
mcp.run(transport="streamable-http", host="0.0.0.0", port=8000, stateless_http=True)
```

**WRONG**:
```python
# stdio transport — local dev only, never production
mcp.run(transport="stdio")

# SSE transport — deprecated, session-bound
mcp.run(transport="sse")

# Missing stateless_http — server creates sticky sessions
mcp = FastMCP("my-server")
```

**Verify**:
```bash
grep -rn "stateless_http=True" src/ && grep -rn 'transport="streamable-http"' src/
```

---

### Law 7: Transport-Appropriate Authentication for HTTP Transports (UPDATED v18.1)

**Rule**: Every HTTP-exposed MCP server MUST implement the correct auth pattern for its deployment context.

**WHY**: Using `OAuthProvider` on a platform-managed server creates a full OAuth authorization server that blocks ALL incoming requests with 401 Unauthorized. The Manus CLI connector does not support dynamic client registration. Platform-managed servers rely on Traefik TLS + network isolation for security.

**3-Pattern Decision Tree:**

| Pattern | When | Code |
|---------|------|------|
| **A: Platform-Managed** | Server behind Manus + Traefik (our standard) | `mcp = FastMCP("server", version="1.0.0", on_duplicate="error")` — NO auth parameter |
| **B: JWTVerifier** | Public server with external IdP | `auth=JWTVerifier(jwks_uri=..., issuer=..., audience=...)` |
| **C: OAuthProvider** | Standalone server (rare) | `auth=OAuthProvider(issuer_url=..., audience=...)` |

**Pattern A — Platform-Managed (CORRECT for Manus + Traefik):**
```python
from fastmcp import FastMCP

# Auth handled at transport level by Manus platform.
# Security: Traefik TLS + mcp-global-network + container isolation.
# DO NOT add OAuthProvider — it will block ALL requests with 401.
mcp = FastMCP(
    "Your-MCP-Server",
    version="1.0.0",
    on_duplicate="error",
)
```

**Pattern B — JWTVerifier (public server with external IdP):**
```python
from fastmcp import FastMCP
from fastmcp.server.auth import JWTVerifier
import os

verifier = JWTVerifier(
    jwks_uri=os.getenv("OAUTH_JWKS_URI"),
    issuer=os.getenv("OAUTH_ISSUER"),
    audience=os.getenv("OAUTH_AUDIENCE"),
)
mcp = FastMCP("Server", version="1.0.0", auth=verifier, on_duplicate="error")
```

**Exceptions**:
- **stdio transport** (local only) — exempt, no network exposure
- **Health endpoints** — exempt, needed for container orchestration

**Verify (platform-managed):**
```bash
# OAuthProvider must NOT be present for platform-managed servers
grep -rn "OAuthProvider" src/ && echo "FAIL — remove OAuthProvider" || echo "PASS"
```

---

## Section 2: Gold Stack v18.1

Complete dependency specification for every MCP server.

### Python Dependencies

```toml
[project]
requires-python = ">=3.14"

dependencies = [
    "fastmcp>=3.0.2,<4",
    "mcp>=1.26.0,<2",
    "uvloop>=0.22.0",
    "orjson>=3.11.0",
    "tenacity>=9.1.0",
    "pybreaker>=1.4.0",
    "httpx>=0.28.0",
    "pydantic>=2.12.0",
    "structlog>=25.5.0",
    "prometheus-client>=0.24.0",
    "opentelemetry-api>=1.39.0",
    "opentelemetry-sdk>=1.39.0",
    "redis>=5.2.0",
    "aiosqlite>=0.20.0",
    "aiofiles>=24.1.0",
    "limits>=3.13.0",
    "aiolimiter>=1.1.0",
]
```

### Infrastructure Versions

| Component | Version | Notes |
|-----------|---------|-------|
| Python | 3.14.3 | Docker: `python:3.14-slim` |
| UV | 0.10.x | Package manager (not pip) |
| Traefik | 3.6.8 | Reverse proxy with auto-TLS, OCSP stapling |
| Redis | 8.x | JSON native, vector set, 87% faster, AGPLv3 |
| microcheck | latest | 75KB healthcheck binary |
| MCP Spec | 2025-11-25 | Protocol specification date (next: June 2026) |
| Governance | AAIF | Under Linux Foundation |

---

## Section 3: 4-Tier Architecture v3.0

### Architecture Diagram

```
┌─────────────────────────────────────────────────────────┐
│                 TIER 1: PRESENTATION                     │
│  FastMCP 3.0 — Components, Providers, Transforms         │
│  Tools | Resources | Prompts | Templates | Apps | Tasks   │
│  DecoratorProvider | OpenAPIProvider | ProxyProvider       │
│  namespace() | filter() | secure() | version() | rename() │
├─────────────────────────────────────────────────────────┤
│                 TIER 2: LOGIC (Core)                      │
│  API Client (Singleton __new__) | Quality Gate            │
│  Rate Limiter | Circuit Breaker | Adaptive Delivery       │
│  Retry (tenacity)                                         │
├─────────────────────────────────────────────────────────┤
│                 TIER 3: INTELLIGENCE (Optional)           │
│  Validation | Recommender | Error Analysis                │
│  Elicitation (SEP-1330) | Sampling with Tools (SEP-1577) │
├─────────────────────────────────────────────────────────┤
│                 TIER 4: DATA                              │
│  Redis 8 (JSON native, vector set) | SQLite (aiosqlite)  │
│  Prometheus (metrics) | Audit Log (sanitized)             │
│  FastMCP 3.0 Native OTEL (zero-config tracing)           │
└─────────────────────────────────────────────────────────┘
```

### Tier 1 — PRESENTATION

The public surface of the MCP server.

- **FastMCP 3.0 server instance** — the entry point for all MCP protocol interactions
- **Handlers**: Tools (callable functions), Resources (data endpoints), Prompts (template-driven generation)
- **Provider selection**:
  - `DecoratorProvider` — custom tools defined with `@mcp.tool()` decorators
  - `OpenAPIProvider` — auto-generate tools from REST API OpenAPI specs
  - `ProxyProvider` — wrap and expose existing MCP servers
  - `FileSystemProvider` — serve files and directories
- **Transform chains** (composable, stackable):
  - `namespace()` — prefix tool names for multi-server composition
  - `filter()` — expose a subset of tools based on criteria
  - `secure()` — enforce authentication and authorization
  - `version()` — API versioning for backward compatibility
  - `rename()` — rename components

### Tier 2 — LOGIC (Core)

Business logic and resilience patterns. Every server needs this tier.

- **API Client Singleton** — `__new__` pattern (Law 1), lazy init (Law 2), async wrapping (Law 3)
- **Quality Gate** — noise filtering, response normalization, field selection
- **Adaptive Delivery** — response size strategy:
  - Small (<100KB): inline in response
  - Medium (<1MB): summarized with key fields
  - Large (>1MB): reference-stored, return URI
- **Circuit Breaker** — `pybreaker`, `fail_max=5`, `reset_timeout=60`
- **Retry** — `tenacity`, exponential backoff with jitter
- **Rate Limiter** — per-tool category limits (read vs. write vs. bulk)

### Tier 3 — INTELLIGENCE (Optional)

Advanced MCP protocol capabilities. Opt-in per server.

- **Validation** — input validation and business rule enforcement
- **Recommender** — intelligent suggestions based on context
- **Error Analysis** — root cause identification and actionable error messages
- **Elicitation** (SEP-1330) — interactive parameter gathering via `ctx.elicit()`. URL mode, enum schema. The server asks the LLM for missing or ambiguous inputs before executing.
- **Sampling** — LLM-in-the-loop processing via `ctx.sample()`. The server delegates reasoning to the connected LLM.
- **Sampling with Tools** (SEP-1577) — agentic sub-workflows where the LLM can call tools during sampling, enabling multi-step autonomous operations.

### Tier 4 — DATA

Persistence, caching, observability.

- **Redis 8** — distributed cache with:
  - JSON native type (`JSON.SET` / `JSON.GET`) for structured data
  - Vector sets (`VADD` / `VSIM`) for semantic caching and similarity search
  - Bloom filters for deduplication
- **SQLite** (`aiosqlite`) — local persistence for history, audit trails, and offline fallback
- **Prometheus** — metrics exposition (`prometheus-client`), scraped by monitoring stack
- **Audit Log** — automatic sensitive data sanitization (PII, credentials, tokens stripped before write)
- **FastMCP 3.0 Native OTEL** — automatic distributed tracing for all MCP operations with zero manual instrumentation

---

## Folder Structure

Standard layout for every MCP server project.

```
mcp-{server-name}/
├── pyproject.toml              # Gold Stack v18.1 deps
├── Dockerfile                  # python:3.14-slim + UV + microcheck
├── docker-compose.yml          # Traefik labels, mcp-global-network
├── .env                        # Secrets (never committed)
├── .env.example                # Template with descriptions
├── src/
│   └── mcp_{server_name}/
│       ├── __init__.py         # uvloop.install()
│       ├── __main__.py         # Entry point: mcp.run()
│       ├── server.py           # FastMCP instance + lifespan
│       ├── config.py           # Pydantic BaseSettings
│       ├── core/
│       │   ├── api_client.py   # Singleton (Law 1)
│       │   ├── quality_gate.py # Noise filtering
│       │   ├── rate_limiter.py # Per-category limits
│       │   └── circuit_breaker.py
│       ├── tools/
│       │   ├── __init__.py
│       │   └── {category}/
│       │       ├── __init__.py
│       │       └── operations.py
│       ├── resources/          # health, metrics, schema
│       ├── prompts/            # Prompt templates
│       └── data/
│           ├── redis_cache.py
│           ├── sqlite_store.py
│           └── audit_log.py
└── testing/
    └── test_mcp.sh             # Endpoint validation
```
