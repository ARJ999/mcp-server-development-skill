> ⚠️ **SUPERSEDED BY v19.1.6** — 2026-04-23. This file is v18.1-era content retained for historical reference. The authoritative v19.1.6 Bible at `https://github.com/ARJ999/MCP-God-Agent-Development-Bible` (branch `main`, flat layout) takes precedence on every rule below. Mapping:
> - Laws + architecture → `00_PHILOSOPHY.md` + `01_INVIOLABLE_LAWS.md` + `02_ARCHITECTURE.md`
> - Server/skill/tool standards → `03_MCP_SERVER_STANDARD.md` + `04_SKILL_STANDARD.md` + `05_TOOL_STANDARD.md`
> - Pairing Contract + 10 Guardrails (G2 removed v19.1.2) → `07_PAIRING_CONTRACT.md`
> - Upgrade Playbook (9 phases + Phase 1.5) → `08_UPGRADE_PLAYBOOK.md`
> - Completion Gate → `v19.0/09_VERIFICATION_PROTOCOL.md`
> - Upstream sync → `v19.0/13_UPSTREAM_SYNC_PROTOCOL.md`
>
> When these reference files contradict the parent `SKILL.md` or v19.0 Bible, the Bible wins.

---

# Validation Reference

Verification commands and checklists for MCP Server v18.1 compliance.

---

## Section 1: Seven Laws Verification Commands

Quick one-liners to verify each law. Run from the server's root directory.

```bash
# Law 1: Singleton with __new__
echo "=== Law 1: Singleton ===" && \
grep -rn "_instance = None" src/ && grep -rn "__new__" src/ && echo "PASS" || echo "FAIL"

# Law 2: Lazy Initialization
echo "=== Law 2: Lazy Init ===" && \
grep -rn "def __init__" src/**/api_client.py | grep -v "initialized" && echo "WARN: Check __init__" || echo "PASS"

# Law 3: Async Executor Wrapping
echo "=== Law 3: asyncio.to_thread ===" && \
grep -rn "run_in_executor" src/ && echo "FAIL: Use asyncio.to_thread" || echo "PASS"

# Law 4: No Duplicate Singletons
echo "=== Law 4: Single Declaration ===" && \
DECL_COUNT=$(grep -rn "APIClient()" src/ | wc -l) && \
[ "$DECL_COUNT" -le 1 ] && echo "PASS ($DECL_COUNT declarations)" || echo "FAIL ($DECL_COUNT declarations)" && \
grep -rn 'on_duplicate="error"' src/ && echo "FastMCP guard: PASS" || echo "FastMCP guard: FAIL"

# Law 5: No **kwargs
echo "=== Law 5: No **kwargs ===" && \
grep -rn "\*\*kwargs" src/tools/ src/*/tools/ && echo "FAIL" || echo "PASS"

# Law 6: Stateless HTTP
echo "=== Law 6: Stateless HTTP ===" && \
grep -rn "stateless_http=True" src/ && echo "PASS" || echo "FAIL" && \
grep -rn 'transport="streamable-http"' src/ && echo "Transport: PASS" || echo "Transport: FAIL"

# Law 7: Transport-Appropriate Auth (3-Pattern)
echo "=== Law 7: Transport-Appropriate Auth ===" && \
# For platform-managed (Manus+Traefik): OAuthProvider must NOT be present
grep -rn "OAuthProvider" src/ && echo "FAIL — OAuthProvider found (remove for platform-managed)" || echo "Platform-Managed: PASS"
# For public servers with external IdP: JWTVerifier must be present
# grep -rn "JWTVerifier" src/ && echo "JWTVerifier: PASS" || echo "JWTVerifier: FAIL"
```

---

## Section 2: Full Validation Script

```bash
#!/bin/bash
# MCP Server v18.1 Validation Script
# Run from server root directory

PASS=0
FAIL=0
WARN=0

pass() { echo "  [PASS] $1"; ((PASS++)); }
fail() { echo "  [FAIL] $1 — $2"; ((FAIL++)); }
warn() { echo "  [WARN] $1 — $2"; ((WARN++)); }

echo "╔══════════════════════════════════════════════╗"
echo "║  MCP Server v18.1 Validation                 ║"
echo "╚══════════════════════════════════════════════╝"

# === Seven Unbreakable Laws ===
echo ""
echo "── Seven Unbreakable Laws ──"

grep -rq "_instance = None" src/ && grep -rq "__new__" src/ && pass "Law 1: Singleton __new__" || fail "Law 1: Singleton" "Missing __new__ pattern"
grep -rq "get_connection\|_ensure_client\|_connect" src/ && pass "Law 2: Lazy init" || fail "Law 2: Lazy init" "No lazy connection method found"
grep -rq "run_in_executor" src/ && fail "Law 3: Executor" "Found run_in_executor (use asyncio.to_thread)" || pass "Law 3: asyncio.to_thread"
grep -rq 'on_duplicate="error"' src/ && pass "Law 4: on_duplicate guard" || fail "Law 4: on_duplicate" "Missing on_duplicate=\"error\""
grep -rq "\*\*kwargs" src/tools/ src/*/tools/ 2>/dev/null && fail "Law 5: No **kwargs" "Found **kwargs in tool handlers" || pass "Law 5: Explicit params"
grep -rq "stateless_http=True" src/ && pass "Law 6: Stateless HTTP" || fail "Law 6: Stateless HTTP" "Missing stateless_http=True"

# Law 7: Transport-Appropriate Auth
# For platform-managed servers: OAuthProvider must NOT be present
grep -rq "OAuthProvider" src/ && fail "Law 7: Auth" "OAuthProvider found — remove for platform-managed servers" || pass "Law 7: Transport-Appropriate Auth"

# === Gold Stack v18.1 ===
echo ""
echo "── Gold Stack v18.1 ──"

[ -f pyproject.toml ] && pass "pyproject.toml exists" || fail "pyproject.toml" "Missing pyproject.toml"
grep -q 'fastmcp>=3' pyproject.toml 2>/dev/null && pass "FastMCP 3.x" || fail "FastMCP" "Not pinned to >=3"
grep -q 'mcp>=1.26' pyproject.toml 2>/dev/null && pass "MCP SDK 1.26+" || fail "MCP SDK" "Not pinned to >=1.26"
grep -q 'requires-python.*3\.14' pyproject.toml 2>/dev/null && pass "Python >=3.14" || fail "Python" "Not requiring 3.14+"
[ ! -f requirements.txt ] && pass "No requirements.txt" || fail "requirements.txt" "Should use pyproject.toml only"

# === Deprecated Patterns ===
echo ""
echo "── Deprecated Pattern Check ──"

grep -rq "fastmcp<3\|fastmcp.*2\." pyproject.toml 2>/dev/null && fail "FastMCP 2.x pin" "Should be >=3.0.2,<4" || pass "No FastMCP 2.x"
grep -rq "python:3\.13" Dockerfile 2>/dev/null && fail "Python 3.13 in Dockerfile" "Should be python:3.14-slim" || pass "No Python 3.13"
grep -rq "six.*law\|six.*unbreakable" src/ 2>/dev/null && fail "Six Laws reference" "Should be Seven Laws" || pass "No Six Laws refs"
grep -rq 'on_duplicate_tools=' src/ 2>/dev/null && fail "on_duplicate_tools" "Renamed to on_duplicate in FastMCP 3.0" || pass "No on_duplicate_tools"

# === Infrastructure ===
echo ""
echo "── Infrastructure ──"

[ -f Dockerfile ] && pass "Dockerfile exists" || fail "Dockerfile" "Missing Dockerfile"
grep -q "python:3.14-slim" Dockerfile 2>/dev/null && pass "Base: python:3.14-slim" || fail "Base image" "Should be python:3.14-slim"
grep -q "microcheck\|httpcheck" Dockerfile 2>/dev/null && pass "microcheck healthcheck" || fail "Healthcheck" "Use microcheck, not Python"
[ -f docker-compose.yml ] && pass "docker-compose.yml exists" || fail "docker-compose.yml" "Missing compose file"
grep -q "traefik.enable=true" docker-compose.yml 2>/dev/null && pass "Traefik labels" || warn "Traefik" "No Traefik labels"
grep -q "mcp-global-network" docker-compose.yml 2>/dev/null && pass "mcp-global-network" || warn "Network" "Not on mcp-global-network"

# === Observability ===
echo ""
echo "── Observability ──"

grep -rq "structlog" src/ && pass "structlog configured" || fail "Logging" "Use structlog, not stdlib logging"
grep -rq "prometheus_client\|prometheus-client" src/ pyproject.toml && pass "Prometheus metrics" || warn "Metrics" "No Prometheus"
grep -rq "opentelemetry" pyproject.toml 2>/dev/null && pass "OTEL SDK" || warn "OTEL" "No opentelemetry in deps"

# === Summary ===
echo ""
echo "╔══════════════════════════════════════════════╗"
printf "║  Results: %d PASS | %d FAIL | %d WARN        ║\n" $PASS $FAIL $WARN
echo "╚══════════════════════════════════════════════╝"

[ $FAIL -eq 0 ] && echo "✓ Server is v18.1 compliant" || echo "✗ Server has $FAIL violations — fix before deployment"
exit $FAIL
```

---

## Section 3: Pre-Deployment Checklist

### Seven Unbreakable Laws

- [ ] Law 1: API client uses `__new__` singleton with `_instance = None` and `_initialized = False`
- [ ] Law 2: Connections created lazily on first use
- [ ] Law 3: All sync calls wrapped with `asyncio.to_thread()`
- [ ] Law 4: Single `api_client = APIClient()` declaration + `on_duplicate="error"`
- [ ] Law 5: Every tool parameter explicitly typed — zero `**kwargs`
- [ ] Law 6: `stateless_http=True` + `transport="streamable-http"`
- [ ] Law 7: Transport-appropriate auth applied (Platform-managed: no OAuthProvider; Public: JWTVerifier)

### FastMCP 3.0 Configuration

- [ ] `FastMCP()` constructor with `on_duplicate="error"`
- [ ] `mcp.run(transport="streamable-http", host="0.0.0.0", port=8000, stateless_http=True)`
- [ ] Lifespan context manager with 4-tier initialization
- [ ] Tool names match `^[a-zA-Z0-9_]{1,64}$` (SEP-986)

### Transport-Appropriate Authentication (Law 7)

- [ ] Platform-managed (Manus+Traefik): `auth=None` (default), NO OAuthProvider
- [ ] Public servers: `JWTVerifier` with external IdP (JWKS URI, issuer, audience)
- [ ] Verify: `grep -rn "OAuthProvider" src/` returns ZERO matches for platform-managed
- [ ] See `05_SECURITY/02_AUTHENTICATION_PATTERNS.md` for complete guidance

### Data Tier

- [ ] Redis 8 cache with JSON native type
- [ ] SQLite store with WAL mode
- [ ] Prometheus metrics defined
- [ ] Audit log with sanitization

### Observability

- [ ] structlog with JSON output
- [ ] FastMCP 3.0 native OTEL (no manual spans needed)
- [ ] OTEL_EXPORTER_OTLP_ENDPOINT configured
- [ ] Health resource: `health://status`
- [ ] Metrics resource: `metrics://prometheus`

### Infrastructure

- [ ] Dockerfile: `python:3.14-slim` + UV + microcheck
- [ ] docker-compose.yml: Traefik labels + mcp-global-network
- [ ] .env with all secrets (never committed)
- [ ] .gitignore includes .env, __pycache__, .venv
- [ ] No requirements.txt (pyproject.toml only)

### Post-Deployment Verification

```bash
# 1. Container running?
docker ps | grep mcp-server-name

# 2. Container healthy?
docker inspect --format='{{.State.Health.Status}}' mcp-server-name

# 3. Traefik detected?
curl -s http://localhost:8080/api/http/routers | jq '.[] | select(.name | contains("server-name"))'

# 4. HTTPS accessible?
curl -sf https://server-name.arjtech.in/health && echo "OK" || echo "FAIL"

# 5. MCP tools responding?
curl -sf https://server-name.arjtech.in/mcp -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}' | jq '.result.tools | length'

# 6. Metrics available?
curl -sf https://server-name.arjtech.in/mcp -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"resources/read","params":{"uri":"metrics://prometheus"}}' | head -20

# 7. Monitoring auto-discovered? (should appear within ~60s of deploy)
# No manual Prometheus config needed — discovery script reads container HEALTHCHECK
docker exec prometheus wget -qO- 'http://localhost:9090/api/v1/targets?scrapePool=blackbox-mcp-health' 2>/dev/null \
  | python3 -c "import json,sys; [print(t['labels']['instance']) for t in json.load(sys.stdin)['data']['activeTargets'] if 'server-name' in t['labels'].get('instance','')]"
```

---

## Section 4: Bible Source Reference (4 permitted files ONLY)

> **The SKILL.md and this validation reference are sufficient for all standard work.** Only these 4 Bible files may be read — and only when the specific need arises. All other Bible files (DEVELOPER_DIRECTIVE.md, CLAUDE.md, MCP_SERVER_DEVELOPMENT_GUIDE.md, and all chapter files) must NOT be read.

| Need | File (at `/root/aj-workspace/outputs/documents/MCP-God-Agent-Development-Bible/`) |
|------|------|
| Dockerfile/docker-compose/server.py templates | `templates/gold-standard-python/` |
| Troubleshooting a specific error not in Common Mistakes | `07_TROUBLESHOOTING/01_COMMON_ISSUES.md` |
| Redis 8 cache pattern details | `03_EXTERNAL_INTEGRATION/05_REDIS_CACHE_PATTERNS.md` |
| Auth pattern edge cases | `05_SECURITY/02_AUTHENTICATION_PATTERNS.md` |

**Bible repository**: `https://github.com/ARJ999/MCP-God-Agent-Development-Bible`
