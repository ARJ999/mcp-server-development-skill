> ⚠️ **SUPERSEDED BY v19.1.1** — 2026-04-22. This file is v18.1-era content retained for historical reference. The authoritative v19.1.1 Bible at `https://github.com/ARJ999/MCP-God-Agent-Development-Bible` (branch `v19.0`) takes precedence on every rule below. Mapping:
> - Laws + architecture → `v19.0/00_PHILOSOPHY.md` + `01_INVIOLABLE_LAWS.md` + `02_ARCHITECTURE.md`
> - Server/skill/tool standards → `v19.0/03_MCP_SERVER_STANDARD.md` + `04_SKILL_STANDARD.md` + `05_TOOL_STANDARD.md`
> - Pairing Contract + 11 Guardrails → `v19.0/07_PAIRING_CONTRACT.md`
> - Upgrade Playbook (9 phases + Phase 1.5) → `v19.0/08_UPGRADE_PLAYBOOK.md`
> - Completion Gate → `v19.0/09_VERIFICATION_PROTOCOL.md`
> - Upstream sync → `v19.0/13_UPSTREAM_SYNC_PROTOCOL.md`
>
> When these reference files contradict the parent `SKILL.md` or v19.0 Bible, the Bible wins.

---

# Development Workflow Reference

Workflows for building new MCP servers and upgrading existing ones to Bible v18.1 standards.

---

## Section 1: New Server Development — 10-Phase Methodology

### Phase 0: Requirements Analysis

1. **Identify target platform/API** — Determine the external service (e.g., Salesforce, Snowflake, Jira, GitHub).
2. **Map API endpoints to MCP tools** — Enumerate available REST/GraphQL/SDK endpoints and decide which become tools.
3. **Determine authentication pattern (Law 7 — 3-Pattern Decision Tree)**:
   - **Pattern A: Platform-Managed** — Server behind Manus + Traefik → `auth=None` (default). NO OAuthProvider.
   - **Pattern B: JWTVerifier** — Public server with external IdP → `auth=JWTVerifier(...)`.
   - **Pattern C: OAuthProvider** — Standalone server (rare) → `auth=OAuthProvider(...)`.
4. **Choose Provider type**:
   - `DecoratorProvider` — Custom tool logic (most common)
   - `OpenAPIProvider` — Auto-generate tools from OpenAPI spec
   - `ProxyProvider` — Wrap an existing MCP server
5. **Define tool categories** — Group tools for rate limiting (e.g., `read`, `write`, `admin`, `bulk`).

### Phase 1: Project Scaffold

1. Create project directory: `mcp-{server-name}/`
2. Initialize with UV:
   ```bash
   uv init --python 3.14
   ```
3. Create the 4-Tier folder structure (see `canonical-standards.md`):
   ```
   mcp-{server-name}/
   ├── src/
   │   └── mcp_{server_name}/
   │       ├── __init__.py          # uvloop.install()
   │       ├── __main__.py          # mcp.run(transport="streamable-http", stateless_http=True)
   │       ├── server.py            # FastMCP instance + lifespan
   │       ├── config.py            # Pydantic BaseSettings
   │       ├── client.py            # API client singleton (Tier 2)
   │       ├── tools/               # Tier 1 — tool handlers
   │       │   ├── __init__.py
   │       │   └── {category}/
   │       ├── resources/           # Tier 1 — resource handlers
   │       ├── prompts/             # Tier 1 — prompt templates
   │       └── data/                # Tier 4 — cache, store, metrics
   ├── pyproject.toml
   ├── Dockerfile
   ├── docker-compose.yml
   ├── .env.example
   └── .gitignore
   ```
4. Write `pyproject.toml` with Gold Stack v18.1 dependencies.
5. Write `.env.example` with all required environment variables.
6. Write `.gitignore` (include `.env`, `__pycache__/`, `.venv/`, `*.pyc`).

### Phase 2: Server Foundation

Create `src/mcp_{server_name}/server.py`:

```python
from fastmcp import FastMCP

# Pattern A: Platform-Managed Auth (Law 7)
# DO NOT add OAuthProvider for Manus+Traefik servers.
mcp = FastMCP(
    "server-name",
    version="1.0.0",
    on_duplicate="error",         # Fail fast on conflicts
)
```

Key files:

| File | Purpose |
|---|---|
| `server.py` | FastMCP instance, lifespan context manager, auth config (Law 7) |
| `__init__.py` | `uvloop.install()` at import time |
| `__main__.py` | `mcp.run(transport="streamable-http", stateless_http=True)` entry point |
| `config.py` | Pydantic `BaseSettings` for all configuration |

- Implement lifespan context manager for startup/shutdown resource management.
- Configure transport-appropriate auth (Law 7) based on deployment context.

### Phase 3: API Client (Tier 2)

Implement the singleton API client in `client.py`:

1. **Singleton via `__new__`** (Law 1) — Not a `_instances` dictionary pattern.
2. **Lazy connection initialization** (Law 2) — Never connect in `__init__`; connect on first use.
3. **Async wrapping** (Law 3) — Wrap all synchronous SDK calls with `asyncio.to_thread()`.
4. **Single declaration** (Law 4) — One instance at module bottom, imported everywhere.
5. **Connection pooling** — Use `httpx.AsyncClient` with connection limits.
6. **Circuit breaker** — Use `pybreaker` to prevent cascading failures.
7. **Retry logic** — Use `tenacity` with exponential backoff.

```python
class PlatformClient:
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

    async def connect(self):
        """Lazy initialization — called on first use, not in __init__."""
        if not self._connected:
            # Initialize connection pool, authenticate, etc.
            ...

client = PlatformClient()  # Single declaration
```

### Phase 4: Tool Implementation (Tier 1)

Create tool handlers under `src/tools/{category}/`:

Each tool must implement:

| Requirement | Detail |
|---|---|
| Explicit typed parameters | Law 5 — NO `**kwargs` in any tool handler |
| Rate limiting decorator | Per-category limits (e.g., 100/min for reads, 20/min for writes) |
| Quality Gate filtering | Return only relevant fields, not raw API responses |
| Adaptive Delivery | Small (inline), medium (structured), large (paginated/summarized) |
| Error handling | Structured error responses with actionable messages |
| Audit logging | Log tool invocations with sanitized parameters |
| SEP-986 naming | Tool names match `^[a-zA-Z0-9_]{1,64}$` |

### Phase 5: Resources and Prompts (Tier 1)

**Required resources:**

- `health://status` — Server health check (connectivity, uptime, version)
- `metrics://prometheus` — Prometheus-format metrics export

**Domain-specific resources** (as needed):

- Schema resources (e.g., `schema://objects`, `schema://fields/{object}`)
- Configuration resources (e.g., `config://limits`, `config://features`)

**Prompt templates:**

- Common workflow prompts for the target platform
- Parameterized templates with clear descriptions

### Phase 6: Data Tier (Tier 4)

| Component | Technology | Purpose |
|---|---|---|
| Cache | Redis 8 with JSON native type | `JSON.SET` / `JSON.GET` for structured caching |
| Persistence | SQLite via `aiosqlite` (WAL mode) | History, audit trail, local state |
| Metrics | Prometheus client | Tool calls, latency histograms, cache hit rates, circuit breaker state |
| Audit log | Structured logging | All tool invocations with sensitive data sanitized |

### Phase 7: Security (Law 7 — Transport-Appropriate Auth)

1. **Determine auth pattern** based on deployment context:
   - **Platform-managed** (Manus+Traefik): `auth=None`. NO OAuthProvider.
   - **Public server**: `JWTVerifier` with external IdP.
   - **Standalone** (rare): `OAuthProvider`.
2. **Input validation** — Pydantic models for all tool parameters.
3. **Output sanitization** — Strip secrets, tokens, and PII from responses.
4. **Secrets management** — All credentials in `.env`, never in source code.
5. **Verify**: `grep -rn "OAuthProvider" src/` returns ZERO matches for platform-managed servers.

### Phase 8: Observability

FastMCP 3.0 provides automatic OpenTelemetry tracing with zero configuration.

1. Set `OTEL_EXPORTER_OTLP_ENDPOINT` environment variable (e.g., `http://127.0.0.1:4317`).
2. **Do not** add manual OTEL instrumentation for tool calls — FastMCP handles this natively.
3. Add custom spans only for business logic that warrants additional visibility.
4. Configure `structlog` with OTEL trace correlation for log-to-trace linking.
5. Define Prometheus metrics for domain-specific KPIs.

### Phase 9: Deployment

1. **Dockerfile:**
   ```dockerfile
   FROM python:3.14-slim
   # Install UV, copy project, uv sync
   # Use microcheck for healthcheck
   HEALTHCHECK CMD ["httpcheck", "http://localhost:8000/health"]
   ```
2. **docker-compose.yml:**
   - Traefik labels for automatic HTTPS routing (`{name}.arjtech.in`)
   - Attach to `mcp-global-network`
   - Set resource limits (memory, CPU)
3. **Build and deploy:**
   ```bash
   docker compose up -d --build
   ```
4. **Verify:**
   - Health endpoint returns 200
   - Traefik routing resolves correctly
   - HTTPS certificate is valid
   - MCP tools respond via streamable-http
   - **Monitoring auto-discovered** — Mission Control picks up the server automatically via the Dockerfile `HEALTHCHECK`. No manual Prometheus config needed. The HEALTHCHECK command determines classification: HTTP health endpoints (`/health`, `/healthz`) → HTTP probe; socket/TCP checks → TCP probe. Verify the server appears on `mc.arjtech.in` MCP Fleet within ~60s of deployment.

---

## Section 2: Upgrade Existing Server Workflow

> ⚠️ **PRODUCTION CONTEXT**: All MCP servers at `/opt/mcp-servers/` are LIVE PRODUCTION systems — currently functioning, well-configured, serving real workloads. Every `.env` contains meticulously configured credentials. **Losing even ONE env var = production disruption.** Treat every upgrade as surgery on a running system.

### Upgrade = Two Mandatory Tracks

Every MCP server upgrade has exactly two dimensions. Both are mandatory. Neither is optional.

| Track | What | Source of Truth | Research Method |
|-------|------|----------------|-----------------|
| **Technical** | Bible compliance, dependencies, architecture, infrastructure | This framework's templates + canonical standards | Read skill files. **NO web search, NO looking at other servers** |
| **Functional** | New API endpoints → new tools, deprecated APIs → removed tools | Platform vendor's API documentation | WebSearch/WebFetch for vendor docs. This is the ONLY permitted external research |

**Planning discipline — what to research and what NOT to research:**
- Read the server's existing source code to understand current state
- Read this framework's templates for target technical state
- Research the platform vendor's API docs for new/changed/deprecated endpoints
- **NEVER** look at other deployed MCP servers (e.g., kaggle-mcp, salesforce-mcp) for "how they did it"
- **NEVER** search the web for FastMCP/MCP framework patterns — this skill has everything

### Step 1: Preserve Existing State + Audit

**BEFORE touching ANY code, capture the complete current state:**

```bash
# 1. CRITICAL: Read and preserve ALL credentials
cat .env                           # Copy EVERY line — API keys, tokens, secrets, connection strings
cat docker-compose.yml             # Capture ALL env vars, networks, labels, volumes, ports

# 2. Audit current architecture
grep "fastmcp\|mcp\|python" pyproject.toml
grep "FROM python:" Dockerfile
grep "stateless_http\|on_duplicate" src/*/server.py
```

Document:
- **Every environment variable and credential** — verbatim, nothing omitted
- Current architecture version
- Tool count, resource count, prompt count
- Deprecated patterns in use (e.g., `run_in_executor`, `_instances` dict, `**kwargs`, `on_duplicate_tools`, `OAuthProvider` on platform-managed)
- If unsure whether a variable is still needed — **keep it and ask the user**

### Step 2: Deep-Research Platform APIs (Functional Track)

> This step covers the **Functional Track** — researching the platform vendor's API for new capabilities. This is the ONLY external research permitted during upgrade. Technical patterns come from this framework, not the web.

1. Research ALL latest APIs and features from the platform vendor (GA + public preview) using WebSearch/WebFetch.
2. Map every existing tool against the latest API surface.
3. Identify:
   - **Gaps** — New API capabilities not yet exposed as tools.
   - **Deprecated endpoints** — APIs retired by the vendor.
   - **New capabilities** — Features that enable better tools.
4. Document findings before making any changes.

### Step 3: Update Dependencies

Update `pyproject.toml` to Gold Stack v18.1, then sync:

```bash
uv sync
```

**FastMCP 3.0 migration — breaking changes to address:**

| Area | FastMCP 2.x / Old v18.0 | FastMCP 3.0 / v18.1 |
|---|---|---|
| Duplicate tools param | `on_duplicate_tools="error"` | `on_duplicate="error"` (renamed) |
| Auth (platform-managed) | `OAuthProvider(...)` | `auth=None` — NO OAuthProvider |
| Auth (public server) | `OAuthProvider(...)` | `auth=JWTVerifier(...)` |
| Async state methods | `ctx.get_state()` (sync) | `await ctx.get_state()` (async) |
| Constructor kwargs | Various kwargs accepted | Removed — use config object |

### Step 4: Apply Seven Unbreakable Laws

Verify and fix each law in order:

| Law | Check | Fix |
|---|---|---|
| 1. Singleton | Uses `__new__` method + `_initialized = False` | Replace `_instances` dict with `__new__` |
| 2. Lazy Init | No connection in `__init__` | Move connection logic to `connect()` method |
| 3. Async | No `run_in_executor` calls | Replace with `asyncio.to_thread()` |
| 4. Single Declaration | One client instance + `on_duplicate="error"` | Consolidate to module-level singleton |
| 5. Explicit Params | No `**kwargs` in tool handlers | Replace with explicit typed parameters |
| 6. Stateless HTTP | `stateless_http=True` in `mcp.run()` | Add to run() call |
| 7. Transport-Appropriate Auth | Correct pattern for deployment | Platform-managed: remove OAuthProvider; Public: add JWTVerifier |

### Step 5: Upgrade Architecture to v3.0

- Add lifespan context manager if missing.
- Implement Quality Gate pattern — filter API responses to relevant fields only.
- Implement Adaptive Delivery pattern — response size determines format.
- Add rate limiting per tool category.
- Add circuit breaker (`pybreaker`) and retry (`tenacity`).
- Convert to Providers/Transforms pattern if server wraps multiple APIs.

### Step 6: Upgrade Data Tier

| Component | Old Pattern | New Pattern |
|---|---|---|
| Redis cache | String SET/GET | Redis 8 JSON native (`JSON.SET` / `JSON.GET`) |
| Vector search | N/A | Redis vector sets (`VADD` / `VSIM`) if applicable |
| SQLite | sync sqlite3 | `aiosqlite` with WAL mode |
| Metrics | Absent or ad-hoc | Prometheus client with standard metric names |
| Audit log | Absent or unstructured | Structured log with sensitive data sanitization |

### Step 7: Upgrade Observability

1. **Remove** all manual OTEL instrumentation — FastMCP 3.0 handles tracing natively.
2. Set `OTEL_EXPORTER_OTLP_ENDPOINT` in `.env`.
3. Add custom spans only for business logic that needs additional visibility.
4. Configure `structlog` with OTEL trace correlation.

### Step 8: Upgrade Infrastructure

1. **Dockerfile** — Update to `python:3.14-slim`, UV for dependency management, `microcheck` for healthcheck. The `HEALTHCHECK` command drives monitoring auto-discovery — use `httpcheck`/`microcheck` with `/health` or `/healthz` for HTTP monitoring; socket-based checks get TCP-only monitoring.
2. **docker-compose.yml** — Traefik 3.6.8 labels, resource limits, `mcp-global-network`.
3. **Remove** `requirements.txt` — Use `pyproject.toml` + `uv.lock` exclusively.

### Step 9: Add New Capabilities

Based on findings from Step 2:

1. **Add** new tools for newly discovered API endpoints.
2. **Remove** tools for deprecated or retired APIs — zero dead code.
3. **Add MCP Apps** (SEP-1865) if interactive UI is needed.
4. **Add MCP Tasks** (SEP-1686) if long-running operations exist.
5. **Rename** tools to SEP-986 compliance: `^[a-zA-Z0-9_]{1,64}$`.

### Step 10: Restore Credentials, Validate, and Deploy

1. **Restore ALL credentials** — Diff the old `.env` against the new `.env` line by line. Every API key, token, secret, and connection string from the original MUST be present. If anything is missing, the upgrade is NOT ready.
2. Run Seven Laws verification commands (grep for each violation pattern).
3. Run full validation script (health, tools listing, sample invocations).
4. Build and deploy:
   ```bash
   docker compose up -d --build
   ```
5. **Verify server starts cleanly** — check logs for auth errors, missing credentials, connection failures.
6. Verify all endpoints (health, Traefik routing, HTTPS, tool responses).
7. **Mandatory skill sync** — The skill at `~/.claude/skills/{platform}/` IS the intelligence layer. Without it, the server is a dumb pipe.
   - Query live `tools/list` → compare every tool name + params against the target skill's *references/tool_catalog.md* (path is per-skill, not a pointer to this file) → fix all drift.
   - If skill doesn't exist → create using Snowflake v2.0 pattern (SKILL.md + tool_catalog + task_patterns + decision_logic + quality_gates).
   - Verify: `tools/list` count == the target skill's *tool_catalog.md* count == SKILL.md stated count. Zero drift.
8. **If any credential fails or any tool errors — the upgrade is NOT complete.** Fix before proceeding.
9. **Mandatory GitHub sync** — Sync working VPS code to the server's GitHub repository. If repo URL unknown → **ask user** before proceeding. Procedure:
   ```bash
   # Clone repo to /tmp, replace with working code, push
   gh repo clone {owner}/{repo} /tmp/{repo}
   git config --global --add safe.directory /tmp/{repo}
   cd /tmp/{repo} && git checkout -b upgrade/bible-v18.1
   rsync -av --delete --exclude='.git' --exclude='.env' --exclude='__pycache__' \
     --exclude='*.pyc' --exclude='.venv' --exclude='uv.lock' /opt/mcp-servers/{server}/ .
   # Audit for secrets: grep -rn "secret\|password\|api_key" --include="*.py" | grep -v "env\|example"
   git add -A && git commit && git push -u origin upgrade/bible-v18.1
   gh pr create --title "..." --body "..."
   # Cleanup
   git config --global --unset safe.directory /tmp/{repo}
   rm -rf /tmp/{repo}
   ```
   The upgrade is NOT complete until the PR exists. This is the backup and the public face.

---

## Section 3: MCP Quality Audit Integration

The MCP Server Development skill covers **building and upgrading** the server itself. Post-deployment auditing of a server + skill combination is handled by the **MCP Quality Audit protocol**.

When the user says `quality audit {connector}`, follow the full 12-phase protocol defined in MEMORY.md:

| Phases | Scope |
|---|---|
| 0-2 | **Discovery** — Container, IP, URL, skill path, server source code |
| 3-5 | **Server-side audit** — Bugs, error messages, dead code |
| 6-8 | **Skill docs audit** — Parameters, descriptions, coverage, workflows |
| 9-10 | **Remediation** — Fix server code + skill docs, rebuild container |
| 11 | **Report** — Archive to `outputs/reports/` |

**Boundary:** Development Workflow handles creation/upgrade. Quality Audit handles post-deployment verification and remediation. No overlap between the two.
