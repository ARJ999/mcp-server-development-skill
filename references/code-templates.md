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

# Code Templates — MCP Server v18.1

> CANONICAL templates. Every new server MUST start from these. No deviations.

---

## Template 1: server.py (Tier 1 — Presentation)

```python
"""MCP Server — {Server Name}."""

from __future__ import annotations

from collections.abc import AsyncIterator
from contextlib import asynccontextmanager

import orjson
import structlog
from fastmcp import FastMCP
from starlette.requests import Request
from starlette.responses import JSONResponse

from .config import settings

# ═══════════════════════════════════════════════════════════════════════
# STRUCTURED LOGGING — orjson serializer (mandatory v18.1)
# ═══════════════════════════════════════════════════════════════════════
structlog.configure(
    processors=[
        structlog.contextvars.merge_contextvars,
        structlog.processors.add_log_level,
        structlog.processors.TimeStamper(fmt="iso"),
        structlog.processors.StackInfoRenderer(),
        structlog.processors.format_exc_info,
        structlog.processors.JSONRenderer(serializer=orjson.dumps),
    ],
    logger_factory=structlog.PrintLoggerFactory(),
)

logger = structlog.get_logger()


@asynccontextmanager
async def lifespan(app: FastMCP) -> AsyncIterator[dict]:
    """Initialize all 4 tiers at startup, cleanup at shutdown."""
    logger.info("server_starting", name=settings.server_name)

    # Tier 2: API Client (warm up lazy connection)
    from .core.api_client import api_client
    try:
        _ = api_client.http  # trigger lazy property
        logger.info("api_client_ready", stats=api_client.get_stats())
    except Exception as e:
        logger.warning("api_client_deferred", reason=str(e))

    # Tier 4: Data (optional — add per server needs)
    # from .data.redis_cache import redis_cache
    # await redis_cache.connect()

    logger.info("server_ready", name=settings.server_name)

    yield {}

    # Cleanup
    await api_client.close()
    logger.info("server_stopped", name=settings.server_name)


# ═══════════════════════════════════════════════════════════════════════
# FASTMCP 3.0 SERVER — Transport-Appropriate Auth (Law 7)
# ═══════════════════════════════════════════════════════════════════════
# Pattern A: Platform-Managed (Manus + Traefik) — DEFAULT
#   Auth handled at transport level by Manus platform.
#   Security: Traefik TLS + mcp-global-network + container isolation.
#   DO NOT add OAuthProvider — it will block ALL requests with 401.
#
# Pattern B: Public server with external IdP:
#   from fastmcp.server.auth import JWTVerifier
#   verifier = JWTVerifier(jwks_uri=..., issuer=..., audience=...)
#   mcp = FastMCP("server", auth=verifier, ...)
#
# Pattern C: Standalone server (rare):
#   from fastmcp.server.auth import OAuthProvider
#   oauth = OAuthProvider(issuer_url=..., audience=...)
#   mcp = FastMCP("server", auth=oauth, ...)

mcp = FastMCP(
    settings.server_name,
    version="1.0.0",
    on_duplicate="error",          # Fail fast on conflicts
    lifespan=lifespan,
)


# ═══════════════════════════════════════════════════════════════════════
# HEALTH CHECK ENDPOINT — Dynamic tool count (mandatory v18.1)
# ═══════════════════════════════════════════════════════════════════════
# NOTE: FastMCP 3.x removed _tool_manager._tools.
#       Use `await mcp.list_tools()` for dynamic tool count.
@mcp.custom_route("/health", methods=["GET"])
async def health_check(request: Request) -> JSONResponse:
    """Health check — returns server status with dynamic tool count."""
    tools = await mcp.list_tools()
    return JSONResponse({
        "status": "healthy",
        "version": "1.0.0",
        "tools": len(tools),
    })


# ═══════════════════════════════════════════════════════════════════════
# TOOL REGISTRATION — Import triggers @mcp.tool() decorators
# ═══════════════════════════════════════════════════════════════════════
from .tools import accounts     # noqa: E402, F401
from .tools import contacts     # noqa: E402, F401
```

---

## Template 2: api_client.py (Tier 2 — Logic)

```python
"""Singleton API Client — Laws 1, 2, 3, 4."""

import asyncio
import httpx
import pybreaker
from tenacity import retry, stop_after_attempt, wait_exponential, retry_if_exception_type
import structlog

from .config import settings

logger = structlog.get_logger()

# Circuit Breaker
api_breaker = pybreaker.CircuitBreaker(
    fail_max=5,
    reset_timeout=60,
    name="api_breaker",
)


class APIClient:
    """Singleton API client with lazy initialization.

    Laws enforced:
        1. Singleton via __new__
        2. Lazy connection on first use
        3. asyncio.to_thread for sync calls
        4. Single instance declaration at module bottom
    """

    _instance = None   # Law 1: Singleton
    _initialized = False

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

    def __init__(self):
        if self._initialized:
            return
        self.__class__._initialized = True

    async def get_connection(self, org_alias: str = "default"):
        """Law 2: Lazy connection — connect on first use."""
        if not hasattr(self, "_http") or self._http is None:
            self._http = httpx.AsyncClient(
                base_url=settings.api_base_url,
                timeout=httpx.Timeout(30.0, connect=10.0),
                limits=httpx.Limits(max_connections=20, max_keepalive_connections=10),
                headers={"Authorization": f"Bearer {settings.api_token}"},
            )
            logger.info("api_client_initialized", org=org_alias)
        return self._http

    @retry(
        stop=stop_after_attempt(3),
        wait=wait_exponential(multiplier=1, min=1, max=10),
        retry=retry_if_exception_type((httpx.ConnectError, httpx.ReadTimeout)),
    )
    async def request(self, method: str, path: str, **kwargs) -> dict:
        """Execute HTTP request with retry + circuit breaker."""
        conn = await self.get_connection()
        # For async HTTP clients, use directly:
        response = await conn.request(method, path, **kwargs)
        response.raise_for_status()
        return response.json()

    async def get(self, path: str, params: dict | None = None) -> dict:
        return await self.request("GET", path, params=params)

    async def post(self, path: str, json: dict | None = None) -> dict:
        return await self.request("POST", path, json=json)

    def get_stats(self) -> dict:
        """Return client stats for health/lifespan logging (mandatory v18.1)."""
        return {
            "client_type": self.__class__.__name__,
            "base_url": str(settings.api_base_url),
            "http_client_active": hasattr(self, "_http") and self._http is not None,
        }

    async def health_check(self) -> dict:
        """Verify API connectivity."""
        try:
            conn = await self.get_connection()
            response = await conn.get("/health")
            return {"status": "ok", "code": response.status_code}
        except Exception as e:
            return {"status": "error", "error": str(e)}

    async def close(self):
        if hasattr(self, "_http") and self._http is not None:
            await self._http.aclose()
            self._http = None


# Law 4: Single declaration — import this everywhere
api_client = APIClient()
```

---

## Template 3: Tool Module (Tier 1 — Presentation)

```python
"""Tool category: {Category Name}."""

import asyncio
import time
import orjson
import structlog

from ...server import mcp
from ...core.api_client import api_client

logger = structlog.get_logger()


@mcp.tool()
async def list_records(
    record_type: str,
    limit: int = 50,
    offset: int = 0,
    org_alias: str = "default",
) -> str:
    """List records of a given type with pagination.

    Args:
        record_type: Type of record to list (e.g., 'account', 'contact').
        limit: Maximum records to return (1-200, default: 50).
        offset: Pagination offset (default: 0).
        org_alias: Organization alias for multi-org support (default: 'default').

    Returns:
        JSON with status, records array, and pagination metadata.
    """
    start = time.perf_counter()
    try:
        logger.info("tool_start", tool="list_records", record_type=record_type)

        result = await api_client.get(
            f"/api/{record_type}",
            params={"limit": limit, "offset": offset},
        )

        # Quality Gate: filter noise, preserve signal
        records = result.get("records", [])

        elapsed_ms = (time.perf_counter() - start) * 1000
        logger.info("tool_success", tool="list_records", count=len(records), ms=round(elapsed_ms, 1))

        return orjson.dumps({
            "status": "success",
            "data": records,
            "pagination": {
                "total": result.get("total", len(records)),
                "limit": limit,
                "offset": offset,
                "has_more": result.get("has_more", False),
            },
        }, option=orjson.OPT_INDENT_2).decode()

    except Exception as e:
        logger.error("tool_error", tool="list_records", error=str(e), error_type=type(e).__name__)
        return orjson.dumps({
            "status": "error",
            "error": str(e),
            "error_type": type(e).__name__,
        }).decode()


@mcp.tool()
async def get_record(
    record_id: str,
    record_type: str,
    org_alias: str = "default",
) -> str:
    """Get a single record by ID.

    Args:
        record_id: Unique record identifier.
        record_type: Type of record (e.g., 'account', 'contact').
        org_alias: Organization alias (default: 'default').

    Returns:
        JSON with full record details.
    """
    try:
        result = await api_client.get(f"/api/{record_type}/{record_id}")
        return orjson.dumps({"status": "success", "data": result}, option=orjson.OPT_INDENT_2).decode()
    except Exception as e:
        return orjson.dumps({"status": "error", "error": str(e)}).decode()
```

---

## Template 4: \_\_init\_\_.py

```python
"""MCP Server — {Server Name}."""
import sys

if sys.platform != "win32":
    try:
        import uvloop
        uvloop.install()
    except ImportError:
        pass
```

---

## Template 5: \_\_main\_\_.py

```python
"""Entry point — streamable-http with stateless mode (Law 6)."""
from .server import mcp

if __name__ == "__main__":
    mcp.run(
        transport="streamable-http",
        host="0.0.0.0",
        port=8000,
        stateless_http=True,
    )
```

---

## Template 6: config.py

```python
"""Server configuration via environment variables."""
from pydantic_settings import BaseSettings
from pydantic import SecretStr


class Settings(BaseSettings):
    # Server
    server_name: str = "my-mcp-server"
    log_level: str = "INFO"

    # API
    api_base_url: str = ""
    api_token: SecretStr = SecretStr("")

    # Redis
    redis_url: str = "redis://redis:6379/0"
    cache_ttl: int = 300

    # OTEL
    otel_endpoint: str = "http://127.0.0.1:4317"

    model_config = {"env_prefix": "MCP_", "env_file": ".env"}


settings = Settings()
```

---

## Template 7: Dockerfile

```dockerfile
# ============================================================================
# MCP Server — Dockerfile v18.1
# God-Grade Architecture v3.0 | Python 3.14 | UV | microcheck
# ============================================================================

FROM python:3.14-slim AS base

# Install UV from official image (10-100x faster than pip)
COPY --from=ghcr.io/astral-sh/uv:latest /uv /usr/local/bin/uv

# Install microcheck for health checks (75KB, zero dependencies)
# Image: ghcr.io/tarampampam/microcheck:1 — binary at /bin/httpcheck
# MANDATORY: Do NOT use Python for health checks
COPY --from=ghcr.io/tarampampam/microcheck:1 /bin/httpcheck /usr/local/bin/httpcheck

WORKDIR /app

# Security: Non-root user with writable home for UV cache
# CRITICAL: Must use -m flag to create home directory, otherwise
# UV fails at runtime with "Permission denied" on /home/mcp/.cache/uv
RUN groupadd -r mcp && useradd -r -g mcp -m mcp

# Copy dependency files first (cache layer optimization)
COPY pyproject.toml uv.lock* ./

# Install dependencies (production only)
RUN uv sync --no-dev --frozen || uv sync --no-dev

# Copy application code
COPY src/ src/

# Set ownership
RUN chown -R mcp:mcp /app

# Switch to non-root user
USER mcp

# Expose Streamable HTTP port
EXPOSE 8000

# HEALTHCHECK: microcheck (NOT Python) - Ultra-lightweight
# interval: 120s - Minimal CPU usage for personal VPS
# start_period: 30s - Allow time for server startup
HEALTHCHECK --interval=120s --timeout=5s --start-period=30s --retries=3 \
    CMD ["httpcheck", "http://localhost:8000/health"]

# Run as module via UV
CMD ["uv", "run", "python", "-m", "src"]
```

---

## Template 8: docker-compose.yml

```yaml
services:
  mcp-server-name:
    build: .
    container_name: mcp-server-name
    restart: unless-stopped
    env_file: .env
    networks:
      - mcp-global-network
    deploy:
      resources:
        limits:
          memory: 512M
          cpus: "1.0"
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.mcp-server-name.rule=Host(`server-name.arjtech.in`)"
      - "traefik.http.routers.mcp-server-name.entrypoints=websecure"
      - "traefik.http.routers.mcp-server-name.tls.certresolver=letsencrypt"
      - "traefik.http.services.mcp-server-name.loadbalancer.server.port=8000"
      - "traefik.http.services.mcp-server-name.loadbalancer.healthcheck.path=/health"
      - "traefik.http.services.mcp-server-name.loadbalancer.healthcheck.interval=120s"

networks:
  mcp-global-network:
    external: true
```

---

## Template 9: pyproject.toml

```toml
[project]
name = "mcp-server-name"
version = "1.0.0"
description = "God-Grade MCP Server — Architecture v3.0"
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
    "pydantic-settings>=2.8.0",
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

[tool.ruff]
target-version = "py314"
line-length = 120

[tool.ruff.lint]
select = ["E", "F", "I", "N", "W", "UP"]
```

---

## Template 10: .env.example

```bash
# Server
MCP_SERVER_NAME=my-mcp-server
MCP_LOG_LEVEL=INFO

# API Credentials
MCP_API_BASE_URL=https://api.example.com
MCP_API_TOKEN=your-api-token-here

# Redis 8
MCP_REDIS_URL=redis://redis:6379/0
MCP_CACHE_TTL=300

# OTEL
MCP_OTEL_ENDPOINT=http://127.0.0.1:4317
```
