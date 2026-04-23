---
name: mcp-server-development
description: "Use when developing a new MCP server from scratch, upgrading an existing MCP server, performing a quality audit, or pairing a skill with its MCP. Complete operational reference for the MCP God Agent Development Bible (v19.1.5) — Ten Inviolable Laws, Pairing Contract (12 sections), 10 active Operational Guardrails, 9-Phase Upgrade Playbook with mandatory Phase 1.5 Upstream Capability Sync, Gold Stack pins, 4-Tier v3.1 Architecture, MCP 2025-11-25 spec-compliance (Check 12). Reference pilot: Fivetran (SIGNED, production). Pair-specific artifacts live in each pair's own repo under docs/, not in the Bible."
version: 3.2.1
---

# MCP Server Development — Bible v19.1.5 Framework

Canonical framework for building, upgrading, and auditing god-grade MCP + paired-skill pairs.

> ⚠️ **ABSOLUTE DIRECTIVE — NO EXTERNAL RESEARCH FOR FRAMEWORK DECISIONS**
>
> This skill is the **COMPLETE operational reference** for ALL MCP + skill development and upgrade work.
> - **NEVER** use WebSearch / WebFetch for FastMCP, MCP SDK, MCP protocol, or framework details. The answer is in this skill or the Bible.
> - External web research is PERMITTED ONLY for (a) **platform vendor API documentation** during Phase 1.5 Upstream Capability Sync, and (b) **authoritative upstream release feeds** (GitHub releases, PyPI, official changelogs) during stack currency scans.
> - **Zero deviation. Zero improvisation. Zero lapses.**

## Source of Truth

| Property | Value |
|---|---|
| **Bible Repository** | [ARJ999/MCP-God-Agent-Development-Bible](https://github.com/ARJ999/MCP-God-Agent-Development-Bible) |
| **Authoritative branch** | `main` (flat repo layout post-2026-04-22 restructure) |
| **Framework version** | **v19.1.5** |
| **Canonized** | 2026-04-22 |
| **MCP Spec** | 2025-11-25 |
| **Python SDK** | `mcp >= 1.27.0, < 2` (v2 alpha BANNED in prod) |
| **Reference pilot** | Fivetran — SIGNED, production-deployed, 175 tools at `fivetran-mcp.arjtech.in` |
| **Pair artifacts** | **NEVER in Bible** — always `<pair-repo>/docs/pairing-contract.md`, `<pair-repo>/docs/pilot-learnings.md`, `<pair-repo>/docs/upstream-sync-<date>.md` |

## Two Independent Version Scales (R13)

Every deployed pair reports BOTH scales — do not conflate them:

| Scale | What it tracks | Example |
|---|---|---|
| **Framework version** | Bible documentation version | `Bible v19.1.5` |
| **Server version** | Pair's own app version | `19.1.1` (Fivetran) |

Cascade surfaces per scale:
- **Server** → image tag, `com.<pair>.version` label, `/health.version`, Dockerfile ARG
- **Framework** → `/health.framework`, `com.<pair>.bible` label, `contract.yaml` `framework_version`

Bumping one does NOT force bumping the other. Conflating them = fail-closed at cascade-check.

## Decision Flow

```
┌─────────────────────────────────┐
│ MCP + skill task                │
└────────────────┬────────────────┘
                 ▼
      ┌──────────────────────┐
      │ New / Upgrade / Audit?│
      └──────────────────────┘
         │        │        │
         │        │        └── quality audit → 12-check Completion Gate (§ Verification)
         │        │
         │        └── existing → 9-Phase Upgrade Playbook (§ Upgrade Workflow)
         │                      MANDATORY Phase 1.5 Upstream Capability Sync
         │
         └── new → 10-Phase New Server Methodology
                   (Pairing Contract authored concurrently, stored in
                    <pair-repo>/docs/pairing-contract.md)
```

## Ten Inviolable Laws

| # | Law | Rule | Verify |
|---|---|---|---|
| 1 | **Singleton `__new__`** | API clients use `__new__` + `_instance = None` + `_initialized = False` | `grep -rn "__new__" src/` |
| 2 | **Lazy Init** | Connect on first use, never at import / `__new__` | `grep -rn "get_connection\|_ensure" src/` |
| 3 | **Async Wrapping** | `asyncio.to_thread(fn, *args)` positional; banned: `kwarg=` — use `functools.partial` | no `to_thread(*, kw=v)` |
| 4 | **No Duplicates** | Single `client = Client()` at module bottom | `grep -c "Client()" src/` → 1 per class |
| 5 | **No `**kwargs` in handlers** | Every tool param explicitly typed | `grep -rn "\\*\\*kwargs" src/tools/` → 0 |
| 6 | **Stateless HTTP** | `stateless_http=True` + `transport="streamable-http"` | `grep "stateless_http" src/` |
| 7 | **Transport-Appropriate Auth** | 5-pattern decision (see below) | § Law 7 |
| 8 | **Pairing Contract** | Every production MCP has one paired skill + one 12-section Contract in pair's own repo | nightly drift + setup-curator |
| 9 | **Operational Guardrails** | 10 active guardrails (G1 + G3..G11; G2 REMOVED v19.1.2) | per-guardrail pytest |
| 10 | **Drift Detection** | Event-driven (post-deploy + post-skill-add), NOT nightly cron | `drift-check.sh` + `cascade-check.sh` |

### Law 7 — 5-Pattern Auth Decision Tree

| Pattern | When | Implementation |
|---|---|---|
| **1: stdio** | Local Desktop MCP subprocess | `auth=None` |
| **2: Platform-Managed** (default for VPS) | Traefik + `mcp-global-network` isolation | `auth=None` — **NEVER `OAuthProvider`** (causes blanket 401) |
| **3: JWTVerifier** | Public server with external IdP | `auth=JWTVerifier(jwks_uri=, issuer=, audience=)` |
| **4: OAuthProvider** | Standalone IdP hosted in server (rare) | `OAuthProvider(...)` — CIMD preferred over DCR |
| **5: Client Credentials** | M2M (SEP-1046) | client-credentials flow handler |

⚠️ **CRITICAL**: Adding `OAuthProvider` to platform-managed (Pattern 2) = blanket 401. Historical incident: v18.0→v18.1 fleet-wide outage at commit `a522c03`.

## Pairing Contract (Law 8)

Every production MCP pairs with one skill via a 12-section Contract:

1. Identity Binding | 2. Tool Inventory Sync | 3. Banned-Tool List | 4. Mandatory Pre-Call Sequences | 5. Operational Guardrails | 6. ~~Cost Pre-Flight~~ (REMOVED v19.1.2 — § 6 is empty; block at warehouse layer) | 7. Role/Auth Contract | 8. Optimization Loop Wiring | 9. Shared Gotcha Registry (≥5) | 10. Verification Rituals | 11. Cascade Contract | 12. Invocation Guarantee

**Location**: `<pair-repo>/docs/pairing-contract.md` — NOT in Bible. Bible ships only `templates/pairing-contract-template.md` (blank). R11 rule: single source of truth per pair.

**Invocation Mechanism** (3+1 layers): skill frontmatter → Hookify autoloader registry → event-driven drift-check → (aspirational) server-side `X-Claude-Skill-Loaded` header.

## Ten Active Operational Guardrails (Law 9)

**Core 8 MANDATORY** (apply to every pair):

| # | Name | Purpose | Contract § |
|---|---|---|---|
| G1 | Request tagging | Every upstream call tagged `{agent,session,task,phase}` | 5.request_tagging |
| G3 | Result size cap | Cap LLM-bound results with summary_stats + pagination | 5.result_cap |
| G4 | Statement/request timeout | Hard ceiling; JSON-RPC -32003 on exceed | 5.timeout |
| G6 | Banned-tool block | Contract § 3 list returns -32006 at MCP dispatch | 5.banned_tool_block + 3 |
| G7 | Per-category rate limit | In-memory (stateless pair) OR Redis (replica-shared) | 5.rate_limit |
| G8 | Audit log + sanitization | Secrets/PII redacted; structlog processor chain before emission | 5.audit_log |
| G10 | Circuit breaker + retry | pybreaker 1.4+ + tenacity 9.1+; per upstream method | 5.circuit_breaker |
| G11 | Input validation | Pydantic 2.13+ on every tool argument | 5.input_validation |

**Domain-Conditional 2** (enable per pair-class):

| # | Name | Enable when | Skip when |
|---|---|---|---|
| G5 | Role Least-Privilege | Upstream has role/tier system + multi-tier keys | Single-credential OR no RBAC |
| G9 | Idempotency Tokens | Upstream NOT natively idempotent on writes | Upstream natively idempotent (Fivetran REST) |

**G2 Cost Pre-Flight — REMOVED v19.1.2**: cost attribution belongs at warehouse/analytics layer, not MCP dispatch. Error code `-32007` RESERVED, MUST NOT be reused.

## MCP 2025-11-25 Spec-Compliance (Check 12 — MUST)

Five protocol features, all MUST be implemented per Bible Check 12:

| # | Feature | Mandate | Bible § |
|---|---|---|---|
| **12.1** | Structured output (`outputSchema`) | Every data-returning tool declares a Pydantic output model; `dict/Any` banned | `05 § 4.1` |
| **12.2** | ToolAnnotations | Every tool emits `title` + 4 hints (`readOnlyHint`, `destructiveHint`, `idempotentHint`, `openWorldHint`); icons SHOULD for pairs ≥100 tools | `05 § 4.3` |
| **12.3** | Task annotations | Every tool matching long-running rubric (≥30s or unbounded) declares `task=True`; **R24 (v19.1.5)**: auto-applied `task=True` is suppressed when docket backend is `memory://` | `05 § 4.5`, § 4.5.1 |
| **12.4** | Elicitation registration | If pair has OAuth/consent/secret flows, `FivetranElicitation`-equivalent MUST call `.register(mcp)` | `03 § 6.1` |
| **12.5** | MCP-Protocol-Version header | Server accepts + advertises `MCP-Protocol-Version: 2025-11-25` | spec |

### Task annotation classification rubric (§ 4.5)

| Pattern | `task=True`? |
|---|---|
| `list_*`, `get_*`, `describe_*`, `query_*` | No |
| `create_*` single resource, `modify_*`, `update_*` | No |
| `sync_*` (incremental, typical <30s) | No |
| `trigger_historical_resync`, `backfill_*`, `full_resync_*` | **Yes** |
| `run_transformation`, `run_dbt_job`, `execute_pipeline` | **Yes** |
| `reload_connector_schema` on 1000-table sources | **Yes** |
| `test_*_project`, `setup_*_tests` | **Yes** |
| `export_*`, `bulk_*` operations | **Yes** |

Flipping existing tool to `task=True` is **semantic-breaking** — coordinate with agent-side consumers, bump server semver, document in Gotcha Registry.

### Backend-conditional task gating (R24, v19.1.5) — § 4.5.1 — **MUST**

`task=True` is ONLY semantically meaningful when the Docket backend is an
external dispatcher (`redis://…`, `rediss://…`, or another queue). On the
default `memory://` backend, nothing outside the server process can enqueue or
retrieve tasks — yet the task subsystem still boots a `Worker.run_forever()`
poll loop + cancellation listener that burns ~4% idle CPU per server with zero
functional benefit.

**Rule**: `install_annotator` MUST detect the backend at install time and
suppress auto-applied `task=True` when `settings.docket.url` starts with
`memory://`. Explicit `@mcp.tool(task=True)` still wins — only the
auto-classification path is gated.

**Enable criterion**: set `FASTMCP_DOCKET_URL=redis://…` AND operate in an
environment where external clients/workers actually dispatch via that backend.

**Completion Gate Check 12.3 verification**: when `docket.url == "memory://"`,
`await mcp.get_tasks()` MUST return an empty iterable. A non-empty result on
memory:// = R24 compliance failure.

**Observed impact on v19.1.3-era pairs** (reclaim magnitude ~order):
- Pair A (fivetran-mcp, 177 tools, 5 long-running): 3.63% → 0.23% idle CPU (−94%)
- Pair B (blue-mcp, 129 tools): 4.41% → 0.17% idle CPU (−96%)

See § Pattern 3 for the reference `install_annotator` implementation.

### Input enums (§ 4.6) — SHOULD

Apply where bounded + stable (auth patterns, sync-state enums). Do NOT apply where unbounded (upstream service-type with 100+ entries = maintenance debt).

### Reference pattern (FastMCP 3)

```python
class GetRecordArgs(BaseModel):
    model_config = ConfigDict(extra="forbid", frozen=True)
    record_id: str = Field(..., pattern=r"^[A-Za-z0-9_-]{1,64}$")

class GetRecordResult(BaseModel):
    status: str
    data: dict

@mcp.tool(
    name="get_record",
    title="Get a record by ID",
    readOnlyHint=True,
    destructiveHint=False,
    idempotentHint=True,
    openWorldHint=True,
    outputSchema=GetRecordResult.model_json_schema(),
)
async def get_record(args: GetRecordArgs) -> GetRecordResult:
    """Retrieve a record by identifier.

    GUARDRAILS: G1 tagging, G4 timeout, G7 rate (metadata), G11 validation
    ERRORS: -32602 invalid id, -32003 timeout, -32010 upstream
    """
    result = await asyncio.to_thread(client.get, args.record_id)  # Law 3
    return GetRecordResult(status="success", data=result)
```

## Triad Trigger Rule (R7) — Redis vs stateless pair

A server MAY pair with Redis **if and only if** it hits ONE of three triggers. Otherwise **stateless pair (no Redis, no redis-py dep)** is the correct answer.

| Trigger | Example |
|---|---|
| Shared rate-limit counters across replicas | Snowflake (per-user quota across 3+ pods) |
| Response cache >256KB or >5min TTL | Tableau workbook metadata |
| Pub/sub, queue, or session state | cios-notify |

**Banned pattern** (schrödinger dep, caught in Fivetran Phase 10): keeping `redis-py` import + `RedisCache` module "as fallback" when no trigger applies. Remove the module + the dep.

## Supply-Chain Tier Gating (R16)

| Tier | Context | Required |
|---|---|---|
| **Tier 1** — Personal VPS / R&D | single-operator VPS, internal | TLS + isolation + secret hygiene + G8 audit + trivy scan |
| **Tier 2** — Single-org production | org-wide real data | Tier 1 + SBOM + digest-pin + vuln-scan cron |
| **Tier 3** — Multi-tenant enterprise | external customers, regulated | Tier 2 + cosign + SLSA-L3 + attestation |

Declare in `contract.yaml`:
```yaml
deployment_tier: 1
tier_rationale: "Single-operator VPS; non-customer-facing; internal R&D"
```

**Banned**: requiring cosign/SLSA on a Tier-1 R&D pair (gold-plating). Fivetran pilot is Tier 1.

## R13 Cascade-Discipline — 7-surface atomic version bump

Every version bump MUST update the 7 surfaces atomically. Validated by `cascade-check.sh` (two scales, two expected values):

**Server-version surfaces** (track pair's app version):
1. docker-compose `IMAGE_TAG` default
2. `com.<pair>.version` label
3. `/health.version` response field

**Framework-version surfaces** (track Bible docs version):
4. `com.<pair>.bible` label
5. `contract.yaml` `framework_version`
6. `/health.framework` response field
7. pilot-learnings final-state block (in pair repo)

Commit declaring one version while `/health` emits another = fail-closed.

## R14 Event-Driven Drift Detection (NOT calendar cron)

Stable deployed pairs don't drift daily. `drift-check.sh` invocation pattern:

| Event | Invocation |
|---|---|
| Post-deploy | `drift-check.sh --pair=<slug>` after `docker compose up -d` |
| Post-skill-add/update | `build-autoloader-registry.py && drift-check.sh --pair=<slug>` |
| Pre-merge CI gate | `cascade-check.sh $SERVER $FRAMEWORK $PAIR` |
| Monthly (optional) | 1st of month — catch 35d upstream-sync staleness |

**Banned**: nightly drift-check cron on stable production pairs. Signal-to-noise ≈ 0 for pinned-image pairs.

## Gold Stack (current pins)

```toml
dependencies = [
    "mcp>=1.27.0,<2",             # FastMCP 3.0+ bundled
    "pydantic>=2.13.0",
    "httpx>=0.28.1",
    "uvloop>=0.22.1",
    "orjson>=3.11.8",
    "tenacity>=9.1.4",            # G10
    "pybreaker>=1.4.1",           # G10
    "structlog>=25.5.0",          # G8
    "prometheus-client>=0.25.0",
    "opentelemetry-api>=1.41.0",
    "opentelemetry-sdk>=1.41.0",
    # "redis>=7.4.0",             # ONLY if Triad Trigger hits (R7)
    "aiosqlite>=0.22.0",          # audit-log persistence
    "aiofiles>=25.1.0",
    "aiolimiter>=1.1.0",          # G7 in-memory (stateless pair)
]

[dev-dependencies]
pytest = ">=9.0.3"
pytest-asyncio = ">=1.3.0"
pytest-cov = ">=7.1.0"
ruff = ">=0.15.11"
mypy = ">=1.20.2"
```

**Infrastructure**:
- Python: `python:3.14-slim` (Tier 1) OR `cgr.dev/chainguard/python:3.14.4` (Tier 2+)
- UV: 0.11.7+ · Traefik: 3.6.13 · Redis (if triad-triggered): 8.6.2-alpine
- microcheck: **digest-pinned** (Law 12 Reproducibility; no `:latest` default)

**Supply chain (CI mandatory at Tier 2+)**:
- syft 1.42.4 (SBOM) | cosign 3.0.6 (Tier 3) | trivy 0.70.0 | pip-audit 2.10+ | semgrep 1.160+

Full manifest: Bible `06_STACK_MANIFEST.md`.

## 4-Tier Architecture v3.1

```
┌──────────────────────────────────────────────────────────────┐
│  CROSS-CUTTING — PAIRING LAYER                               │
│  Skill frontmatter ↔ Contract YAML ↔ MCP manifest            │
│  Invocation: frontmatter + Hookify + drift-check + (header)  │
├──────────────────────────────────────────────────────────────┤
│  CROSS-CUTTING — ENFORCEMENT LAYER                           │
│  Contract Enforcer middleware + 10 Operational Guardrails    │
├──────────────────────────────────────────────────────────────┤
│  TIER 1 — PRESENTATION                                       │
│  FastMCP 3, 5+2 primitives (Tools/Resources/Prompts/         │
│  Elicitation/Sampling + Icons/Tasks), Streamable HTTP        │
│  tool annotations + outputSchema MUST                         │
│  TIER 2 — LOGIC                                              │
│  core/: singletons, breaker, retry, rate-limit, tagging,     │
│  result-cap, audit-sanitize, contract-enforcer               │
│  TIER 3 — INTELLIGENCE (optional)                            │
│  Cortex, semantic, ML pre-flight                             │
│  TIER 4 — DATA                                               │
│  Redis 8 (ONLY if Triad Trigger), SQLite audit, metrics      │
└──────────────────────────────────────────────────────────────┘
```

Detail: Bible `02_ARCHITECTURE.md § 1-4`.

## New Server: 10-Phase Methodology

| Phase | Action | Deliverable |
|---|---|---|
| 0 | Requirements + auth pattern (Law 7) + Pairing Contract draft | `<pair-repo>/docs/pairing-contract.md` skeleton |
| 1 | Project scaffold (`uv init`, folder layout per Bible `03 § 9`) | pyproject + src/ |
| 2 | Server foundation (server.py + lifespan + MCP Primitives Registration per `03 § 6`) | Tier 1 skeleton with Resources/Prompts/Elicitation **registered** |
| 3 | API client singleton (Laws 1-4) + circuit breaker + retry | Tier 2 core/ |
| 4 | Tool implementation — each tool MUST have: Pydantic input, Pydantic output (`outputSchema=`), ToolAnnotations, `task=True` if long-running | Tools registered, Check 12.1-12.3 compliant |
| 5 | Resources + prompts + health endpoints + OTel spans | MCP primitives complete |
| 6 | Data tier (Redis ONLY if Triad Trigger + SQLite audit) | Tier 4 |
| 7 | Security (Law 7 pattern + RFC 8707 + session user-binding) | Auth complete |
| 8 | Guardrails — Core 8 mandatory + Domain-Conditional 2 per worksheet + Contract Enforcer | Law 9 live |
| 9 | Deployment (Dockerfile multi-stage + compose + Traefik + SBOM + tier-appropriate signing) | Container in prod, 12-check Gate GREEN |

Pairing Contract authored **concurrently** — by Phase 9, Contract § 1-12 complete in `<pair-repo>/docs/pairing-contract.md` and skill exists at `/root/.claude/skills/<slug>/SKILL.md`.

## Upgrade Workflow — 9-Phase Playbook

> ⚠️ **PRODUCTION CONTEXT**: Every MCP at `/opt/mcp-servers/` is LIVE. Every `.env` has carefully configured credentials. Preserve everything; break nothing. Unsure → keep + ask.

### Pre-flight Rule 0 — `.env` is sacrosanct (MUST, non-negotiable)

The `.env` file at `/opt/mcp-servers/<slug>/.env` contains API credentials, tokens, upstream endpoints, and operator-tuned runtime settings. **BANNED actions during ALL phases of upgrade**:

- Delete `.env` (even if "it looks unused" — you're wrong)
- Rewrite `.env` wholesale (even from a "better template")
- Remove any variable from `.env` without explicit operator confirmation
- Add new variable to `.env` without writing migration note in upgrade commit
- Log `.env` values anywhere (use `.env.example` with placeholder values for templates)

**Rule**: if an env var looks unused in the new code, it MAY be used by a legacy code path you haven't seen, or by an operator override. Default is KEEP. Explicit operator confirmation required before ANY removal.

**Phase 1 — Baseline snapshot**
- Tag current image: `docker tag <image>:<current> <image>:<current>-pre-upgrade`
- Git branch: `backup/<current>-<slug>-<date>`
- Capture live `tools/list` count, SBOM, Redis RDB if persistent
- **Capture `.env` integrity signature** (names + hash, NEVER values):
  ```bash
  mkdir -p /tmp/<slug>-baseline-$(date +%F)
  sha256sum /opt/mcp-servers/<slug>/.env > /tmp/<slug>-baseline-$(date +%F)/env-hash.txt
  grep -oE '^[A-Z_][A-Z0-9_]*=' /opt/mcp-servers/<slug>/.env | sort > \
    /tmp/<slug>-baseline-$(date +%F)/env-vars.txt
  wc -l < /opt/mcp-servers/<slug>/.env > /tmp/<slug>-baseline-$(date +%F)/env-lines.txt
  stat -c '%s %Y' /opt/mcp-servers/<slug>/.env > /tmp/<slug>-baseline-$(date +%F)/env-stat.txt
  ```
- **Gate**: snapshot committed, rollback tag preserved, env-baseline artifacts present

**Phase 1.5 — Upstream Capability Sync (MANDATORY)**
- Run 6-step sync protocol (Bible `13 § 2.3`):
  1. Collect upstream inventory (API / SDK / platform / MCP spec / governance / deprecations)
  2. Snapshot current pair capability (live `tools/list` + SDK pin + Contract § 2)
  3. Diff upstream vs snapshot
  4. Classify as must-adopt / should-adopt / optional / ignored
  5. Author proposal at `<pair-repo>/docs/upstream-sync-<YYYY-MM-DD>.md`
  6. **Review + adopt** — must-adopt items LANDED IN CODE before Phase 2 Contract § 2 frozen
- Update `/root/.claude/skills/<slug>/capability-sync-state.yaml` (`last_synced`, `coverage_ratio`, `next_sync_due`)
- **Zero-deferred-must-adopt rule**: documenting a must-adopt is NOT enough. If it's must-adopt, it ships in this upgrade OR waiver with time-bound expiry. "Noted but not done" is banned.
- **Gate**: sync proposal committed + must-adopts adopted + deprecations resolved (banned in contract OR migrated)

**Phase 2 — Contract draft (12 sections)**
- Copy `templates/pairing-contract-template.md` → `<pair-repo>/docs/pairing-contract.md`
- Fill all 12 sections (see `07_PAIRING_CONTRACT.md`)
- Declare `deployment_tier` (1/2/3) per R16
- Declare guardrail applicability (Core 8 always; G5/G9 per worksheet)
- **Gate**: Contract authoring checklist complete; § 6 empty (G2 removed)

**Phase 3 — Stack bump**
- pyproject.toml per Bible `06_STACK_MANIFEST.md`
- Dockerfile: multi-stage + non-root USER + tier-appropriate base + microcheck digest-pinned
- Phase 3b breaking-change audits (redis-py, limits, major-version SDK bumps)
- Ruff format + check --fix; mypy strict; pytest 9
- **`.env` integrity check** (post-bump; hash MUST match Phase 1 baseline):
  ```bash
  NEW_HASH=$(sha256sum /opt/mcp-servers/<slug>/.env | awk '{print $1}')
  BASELINE_HASH=$(awk '{print $1}' /tmp/<slug>-baseline-$(date +%F)/env-hash.txt)
  [[ "$NEW_HASH" == "$BASELINE_HASH" ]] || { echo "FAIL: .env changed during Phase 3"; exit 1; }
  ```
- **Gate**: all tests green + no accidental dep bumps + `.env` hash-identical to baseline

**Phase 4 — Server-side enforcement**
- Create `src/<pkg>/core/contract_enforcer.py` reading Contract YAML at startup
- Implement Core 8 + Domain-Conditional as applicable
- **MCP Primitives Registration** (`03 § 6`): `resources.register(mcp)`, `prompts.register(mcp)`, `elicitation.register(mcp)` if OAuth/consent flows
- **Tool refactor for Check 12**: each tool gets `outputSchema=`, `ToolAnnotations`, `task=True` if long-running
- pytest: `tests/guardrails/test_g1..g11.py` all green (G2 tests assert `enabled is False` + rationale)
- **Gate**: 10 guardrail tests pass + Check 12 subchecks pass

**Phase 5 — Skill upgrade**
- Frontmatter per `04_SKILL_STANDARD § 2` (`mcp_server`, `mcp_tool_prefix`, `mcp_tool_count`, `mcp_auth`, `mcp_endpoint`, `pairing_contract_path`)
- `pairing_contract_path` = absolute URL to `<pair-repo>/docs/pairing-contract.md` on its default branch
- Nine mandatory sections (`§ 3`) — Identity, Pairing Block, When to Invoke, Tool Map, Pre-Call Sequences, Guardrails mirror, Gotcha Registry (≥5), Verification Rituals, Cascade Notes
- Dual-mirror `/root/.claude/skills/` + `/home/claude/.claude/skills/`
- **R11 Single-source-of-truth**: banned `skill/` subdirectory inside MCP repo
- **Gate**: setup-curator pre-write validator passes

**Phase 6 — Invocation Mechanism wiring**
- Layer 1: skill frontmatter (always)
- Layer 2: `~/.claude/hooks/skill-autoloader-registry.json` (rebuild via `build-autoloader-registry.py`)
- Layer 3: event-driven drift-check (post-deploy + post-skill-add — NOT nightly cron per R14)
- Layer 4: server-side header (aspirational; enable after 3mo stability)
- **Gate**: post-deploy `drift-check.sh --pair=<slug>` clean

**Phase 7 — Packaging + supply chain**
- Tier-appropriate (R16): Tier 1 = trivy scan. Tier 2 = + SBOM + digest-pin. Tier 3 = + cosign + SLSA-L3
- Image tag = SERVER version (NOT framework version)
- **Gate**: tier-required checks all green

**Phase 8 — Staging canary**
- Tier 1 (personal VPS): N/A — accept in Completion Gate as "Check 9 = N/A"
- Tier 2+: deploy to `<slug>-mcp-staging.arjtech.in`, 7-day soak, evaluation harness (`11_EVALUATION_HARNESS.md`) with ≥20pp paired-vs-unpaired gap, p<0.05
- **Gate**: regression-free OR Tier 1 N/A explicit

**Phase 9 — Production cutover**
- `docker compose up -d` with `IMAGE_TAG=v<SERVER_VERSION>` (NOT framework version)
- Run **cascade-check.sh** — 7 surfaces aligned across 2 scales
- **`.env` integrity final check + live credential validation**:
  ```bash
  # (a) .env hash unchanged from Phase 1 baseline
  FINAL_HASH=$(sha256sum /opt/mcp-servers/<slug>/.env | awk '{print $1}')
  BASELINE_HASH=$(awk '{print $1}' /tmp/<slug>-baseline-$(date +%F)/env-hash.txt)
  [[ "$FINAL_HASH" == "$BASELINE_HASH" ]] || { echo "FAIL: .env drifted"; exit 1; }
  # (b) live credentials work — one authenticated call per credential-consuming tool
  #     (Fivetran reference: get_account_info; Snowflake: describe_account;
  #      each pair defines its "identity probe" tool in Contract § 10 Verification Rituals)
  curl -fsS -X POST https://<slug>-mcp.arjtech.in/mcp \
    -H 'Content-Type: application/json' -H 'MCP-Protocol-Version: 2025-11-25' \
    -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"<identity_probe>"}}' \
    | grep -qE '"account_id"|"user_id"|"org_id"' || { echo "FAIL: credentials broken"; exit 1; }
  ```
- Run **12-check Completion Gate** — ALL must pass (or N/A with rationale)
- Mark Contract **SIGNED** (`pair_status: SIGNED` in contract.yaml)
- **Gate**: Completion Gate 12/12 GREEN + `.env` hash-identical + identity probe returns live account

### Rollback procedure — `.env` safety clause

If Phase 9 rollback triggers (any regression within 24h):
- `docker compose up -d` with `IMAGE_TAG=<previous>-pre-upgrade` — image swap ONLY
- **NEVER restore `.env` from a backup** — live `.env` is already correct (it was never touched). Restoring risks overwriting operator edits made between Phase 1 and rollback.
- Container restarts with original image + original `.env` — instant revert to pre-upgrade state

**Phase 10 — Quality audit closure (post-cutover within 48h)**
- Pristine Sweep 7 categories (`09 § 3`) — Sweep 7 is R13 cascade drift
- Capture learnings in `<pair-repo>/docs/pilot-learnings.md`
- Surface framework refinements (`R<N>` numbering) — feed back to Bible if they generalize
- **Gate**: 0 open P1 findings; all findings either resolved or time-bound-waived

## 12-Check Universal Completion Gate

| # | Check | Bible § |
|---|---|---|
| 1 | Ten Laws verified | Bible `01` |
| 2 | Stack pins match manifest | Bible `06` |
| 3 | Pairing Contract SIGNED | Bible `07` |
| 4 | 10 active guardrails tested | Bible `07 § 3` + `09 § 4` |
| 5 | Banned tools return -32006 | Bible `07 § 3` |
| 6 | Skill frontmatter compliant | Bible `04` |
| 7 | Tool inventory sync (live = contract = skill) | Bible `07 § 2` |
| 8 | Supply-chain artifacts present (tier-appropriate) | Bible `06 § 1.6.1` |
| 9 | Staging canary clean | Tier 1 = N/A; Tier 2+ = MUST |
| 10 | Drift detection invocable (event-driven) | Bible `09 Check 10` |
| 11 | Session-close intelligence captured | pilot-learnings.md in pair repo |
| **12** | **MCP 2025-11-25 spec-compliance** (5 subchecks) | Bible `09 Check 12` |

### Check 12 subchecks

| # | Subcheck | Verification |
|---|---|---|
| 12.1 | `outputSchema` on every data-returning tool | `tools/list` — every entry has non-empty outputSchema OR exempt with adoption plan |
| 12.2 | ToolAnnotations — title + 4 hints populated | grep tools/list JSON |
| 12.3 | Task annotations on long-running tools | grep `@mcp.tool(` for rubric matches |
| 12.4 | Elicitation `.register(mcp)` if applicable | `grep -E 'elicitation.*\.register\(mcp\)' src/server.py` |
| 12.5 | `MCP-Protocol-Version: 2025-11-25` accepted + advertised | curl initialize handshake |

**Gate Summary Block** (verbatim in upgrade commit):
```
Completion Gate — <pair-slug>
✓ 1-12 checks per Bible 09_VERIFICATION_PROTOCOL.md
Constraint: Bible v19.1.5 | pairing-contract-v1.0 | deployment-tier: <1|2|3>
Server version: <v>  |  Framework: Bible v19.1.5
Upstream-sync: <date> | Adopted: X/Y must | Coverage: N% | Next-sync: <date>
```

## Pristine Sweep — 7 categories (Bible `09 § 3`)

| # | Category | Check |
|---|---|---|
| 1 | Repo residue | No stale version markers, no TEMPLATE placeholders |
| 2 | Memory drift | MEMORY.md index still accurate |
| 3 | Docs drift | Architecture/onboarding updated if invariant changed |
| 4 | Dependency drift | `uv pip freeze` matches manifest exactly |
| 5 | Test drift | Deleted paths have tests deleted; new guardrails have tests added |
| 6 | Operational drift | Dashboards + alert rules updated |
| 7 | **Cascade drift (R13)** | 7 surfaces aligned atomically across 2 scales (server + framework) |

## Upstream Capability Sync (Phase 1.5 detail)

**Why**: an MCP proxies a moving upstream. Without this protocol, pairs silently fall behind platform capabilities. **Zero-deferred-must-adopt**: if recommended as must-adopt, it ships in the upgrade OR gets a time-bound waiver.

**Three triggers**: scheduled monthly (optional), pre-upgrade (this Phase 1.5), upstream announcement (RSS/webhook).

**Six capability classes**: API endpoints · SDK versions · platform features · MCP spec · **governance endpoints** (RBAC + admin + idempotency; NOT cost since G2 removed — cost → warehouse layer) · deprecations.

**Classification rigour**:
- **must-adopt**: security patch · replacement for deprecated endpoint · MCP-spec-MUST · breaking-migration
- **should-adopt**: net-new capability a senior practitioner would expect
- **optional**: niche endpoints
- **ignored**: off-brand features

**Proposal artifact**: `<pair-repo>/docs/upstream-sync-<YYYY-MM-DD>.md` per template in Bible `13 § 3.3`. Status markers: ✅ DONE · ⚠️ PARTIAL · ❌ OBSOLETED · ⚠️ DEFERRED (with expiry).

**State file**: `/root/.claude/skills/<slug>/capability-sync-state.yaml` (last_synced, coverage_ratio, next_sync_due).

## Check 12 Implementation Playbook — proven patterns (Fivetran pilot reference)

The Fivetran pilot shipped full Check 12 compliance for 175 tools without hand-authoring 175 Pydantic models. The patterns below are **proven, production-tested, and replicable for any pair**.

### Pattern 1 — outputSchema (Check 12.1) via verb-prefix classifier (MANDATORY for pairs >50 tools)

**Don't**: hand-author one Pydantic output model per tool. 175 tools × 30 min each = unshippable.

**Do**: identify your pair's canonical response envelope, create 4-6 variants by tool-class, auto-inject via monkey-patch. Fivetran's upstream REST API wraps every response in `{code, message, data}`; most pairs have a similar shape.

**Reference implementation** (`src/<pair>/core/output_schemas.py`):

```python
from pydantic import BaseModel, Field

class <Pair>Response(BaseModel):
    """Base envelope — every response has code + message + optional data."""
    code: str
    message: str
    data: dict[str, Any] | None = None

class <Pair>ListData(BaseModel):
    items: list[dict[str, Any]] = Field(default_factory=list)
    next_cursor: str | None = None

class <Pair>ListResponse(<Pair>Response):
    """For list_*/search_* — paginated collection."""
    data: <Pair>ListData | None = None

class <Pair>GetResponse(<Pair>Response):
    """For get_*/describe_*/query_* — single resource."""
    data: dict[str, Any] | None = None

class <Pair>MutationResponse(<Pair>Response):
    """For create_/modify_/delete_/update_/add_/remove_/sync_/..."""
    data: dict[str, Any] | None = None

class <Pair>ActionResponse(<Pair>Response):
    """For test_/approve_/run_/pause_/unpause_/revoke_/reload_/cancel_/..."""
    data: dict[str, Any] | None = None

class <Pair>HealthResponse(BaseModel):
    """For health_check / get_server_ / get_prometheus_ / get_component_."""
    status: str
    service: str | None = None
    version: str | None = None
    details: dict[str, Any] | None = None

_RULES = [
    (("list_", "search_"), <Pair>ListResponse),
    (("get_", "describe_", "query_"), <Pair>GetResponse),
    (("create_", "modify_", "update_", "delete_", "add_", "remove_",
      "invite_", "setup_", "sync_", "resync_", "trigger_", "download_",
      "register_"), <Pair>MutationResponse),
    (("test_", "approve_", "run_", "pause_", "unpause_", "revoke_",
      "reload_", "cancel_", "rotate_", "regenerate_", "reset_",
      "upgrade_", "analyze_"), <Pair>ActionResponse),
    (("health_check", "get_server_", "get_prometheus_", "get_component_"),
     <Pair>HealthResponse),
]

def classify_output_schema(name: str) -> dict[str, Any]:
    for prefixes, model in _RULES:
        if any(name == p or name.startswith(p) for p in prefixes):
            return model.model_json_schema()
    return <Pair>Response.model_json_schema()
```

**Fivetran result**: 175/175 tools typed, 6 distinct schemas, zero per-tool boilerplate.

### Pattern 2 — Task annotations (Check 12.3) via LONG_RUNNING_TOOLS frozenset + fastmcp[tasks]

**Dependency MUST**: `fastmcp[tasks]>=3.0.2` (NOT plain `fastmcp`). The `[tasks]` extra pulls `pydocket` + dependencies required for the MCP 2025-11-25 Tasks primitive. Without it, the server crashes at startup with `ImportError: FastMCP background tasks require the 'tasks' extra`.

```toml
# pyproject.toml
"fastmcp[tasks]>=3.0.2,<4",  # [tasks] extra enables MCP 2025-11-25 Tasks primitive
```

Then regenerate `uv.lock`: `uv lock --upgrade-package fastmcp`.

**Reference pattern** (add to `core/output_schemas.py`):

```python
LONG_RUNNING_TOOLS: frozenset[str] = frozenset({
    # Tools with ≥30s wall-clock OR unbounded runtime
    "trigger_historical_resync",    # hours possible on high-MAR sources
    "resync_connector",              # full re-sync
    "resync_connector_table",        # table-scale dependent
    "reload_connector_schema",       # scales with table count
    "run_transformation",            # dbt/Coalesce pipeline run
    "test_transformation_project",   # compile + test
    # Add pair-specific long-running tools here
})

def is_long_running(name: str) -> bool:
    return name in LONG_RUNNING_TOOLS
```

**Backward-compat guarantee**: `task=True` advertises Tasks primitive capability. Clients that opt in (via MCP 2025-11-25 Tasks) get `task_id` + poll. Clients that don't still get synchronous responses. No forced protocol break.

### Pattern 3 — Unified `install_annotator()` — auto-injects all 3 Check 12 fields + R24 backend gate

Single monkey-patch covers Check 12.1 + 12.2 + 12.3 + R24 (v19.1.5). Invoke ONCE right after `mcp = FastMCP(...)`.

```python
# core/tool_annotations.py
def install_annotator(mcp):
    from .output_schemas import classify_output_schema, is_long_running

    # R24 (v19.1.5): task=True is waste on memory:// — no cross-process dispatcher.
    # Detect at install time; explicit @mcp.tool(task=True) still wins over this gate.
    try:
        from fastmcp import settings as _fm_settings
        _docket_url = getattr(_fm_settings.docket, "url", "memory://")
        _tasks_meaningful = not str(_docket_url).startswith("memory://")
    except Exception:
        _tasks_meaningful = False

    original_tool = mcp.tool

    def tool_with_annotations(*args, **kwargs):
        explicit_annotations = kwargs.pop("annotations", None)
        explicit_output_schema = kwargs.pop("output_schema", None)
        explicit_task = kwargs.pop("task", None)

        def decorator(fn):
            annotations = explicit_annotations or classify(fn.__name__)
            output_schema = (
                explicit_output_schema
                if explicit_output_schema is not None
                else classify_output_schema(fn.__name__)
            )
            # R24: auto task=True only when external dispatcher is configured
            if explicit_task is not None:
                task = explicit_task
            elif _tasks_meaningful and is_long_running(fn.__name__):
                task = True
            else:
                task = None
            extra = {"annotations": annotations, "output_schema": output_schema}
            if task is not None:
                extra["task"] = task
            return original_tool(*args, **extra, **kwargs)(fn)

        return decorator

    mcp.tool = tool_with_annotations
```

Explicit kwargs (`annotations=`, `output_schema=`, `task=`) override inference for tools needing custom shapes. The R24 gate affects only the auto-classification path — an explicit `@mcp.tool(task=True)` still activates the Tasks primitive regardless of backend (pair owner's choice).

### Pattern 4 — MCP primitives: register what you use, REMOVE what you don't (R7/R11 extension)

The Fivetran Phase 10 audit surfaced a new ban pattern: **aspirational primitive classes** — classes instantiated (`FivetranResources()`) but never registered with `mcp` and never called by any tool. Same failure mode as R7 schrödinger-dep: exists in code, does nothing, accrues drift.

**Policy** (R7/R11 extension): for each of the 5 MCP primitives (Tools, Resources, Prompts, Elicitation, Sampling):
- If genuinely used by ≥1 tool → keep + register properly
- If zero usages → **DELETE the file entirely**

**Fivetran pilot outcome**:
- `FivetranPrompts`: kept — 3× `render_prompt()` calls in tools
- `FivetranResources`: DELETED — 0 usages; Fivetran exposes data via tools, not Resources primitive
- `FivetranElicitation`: DELETED — Basic auth has no OAuth/consent/secret-entry flow
- `FivetranSampling`: DELETED — doesn't delegate LLM calls to client

### Pattern 5 — Identity probe declaration in Contract § 10

Every pair's `contract.yaml` MUST declare an identity probe tool for Bible Check 3 (c) live credential validation:

```yaml
# contract.yaml § 10 — Verification Rituals
verification_rituals:
  identity_probe:
    tool: <get_account_info | describe_account | describe_organization>
    expected_response_contains: ["account", "id"]  # fields that indicate live auth
    purpose: "Validates <CREDENTIAL_VARS> are live against upstream post-cutover"
```

Post-cutover (Phase 9): call this tool once, verify expected fields present. If auth broken → immediate rollback.

### Pattern 6 — `.env` baseline artifact format

Every pair's `<pair-repo>/docs/env-baseline.txt` MUST document:
- SHA256 hash of `.env` at baseline capture time
- File size + mtime
- List of variable NAMES (values NEVER logged)
- Inline verification command

```yaml
# docs/env-baseline.txt (example)
baseline_captured: 2026-04-22
file:
  path: /opt/mcp-servers/<slug>/.env
  sha256: f349b915b1e56ec4ca769ea1c34f4395b789184e1d68eced60b15c75d1b82529
  size_bytes: 478
variables:
  - FIVETRAN_API_KEY
  - FIVETRAN_API_SECRET
  # ...
verification_command: |
  CURRENT=$(sha256sum /opt/mcp-servers/<slug>/.env | awk '{print $1}')
  [[ "$CURRENT" == "<BASELINE_HASH>" ]] && echo "✓ PRESERVED" || { echo "✗ DRIFT"; exit 1; }
```

Contract.yaml references it:
```yaml
env_integrity:
  baseline_file: docs/env-baseline.txt
  variables: [<list>]
```

---

## FastMCP 3 Critical Patterns (v19.1.5 compliant)

```python
# Server init — Platform-Managed (Law 7 Pattern 2) — default for VPS
# NEVER add OAuthProvider behind Traefik — causes blanket 401
mcp = FastMCP(
    "<slug>",
    version="<SERVER_VERSION>",     # e.g. "1.0.0" — NOT framework version
    on_duplicate="error",           # fail-fast (NOT "on_duplicate_tools")
    lifespan=lifespan,
)

# MCP Primitives Registration (03 § 6 — MUST)
from .mcp_primitives import FivetranResources, FivetranPrompts, FivetranElicitation

resources = FivetranResources();    resources.register(mcp)
prompts = FivetranPrompts();         prompts.register(mcp)
elicitation = FivetranElicitation(); elicitation.register(mcp)  # if OAuth/consent flows

# R15 — per-tool ToolAnnotations via verb-prefix classifier
from .core.tool_annotations import install_annotator
install_annotator(mcp)

# G8 — structlog processor chain with redaction BEFORE JSONRenderer
from .core.sanitize import redact_processor as g8_redact_processor
structlog.configure(processors=[
    structlog.contextvars.merge_contextvars,
    structlog.processors.add_log_level,
    structlog.processors.StackInfoRenderer(),
    structlog.dev.set_exc_info,
    structlog.processors.TimeStamper(fmt="iso"),
    g8_redact_processor,
    structlog.processors.JSONRenderer(serializer=lambda *a, **kw: orjson.dumps(*a, **kw).decode()),
])

# Long-running tool — task=True (12.3)
@mcp.tool(
    task=True,  # MCP 2025-11-25 Tasks primitive — client receives task_id + polls
    outputSchema=HistoricalResyncResult.model_json_schema(),
)
async def trigger_historical_resync(args: HistoricalResyncArgs) -> HistoricalResyncResult:
    """Long-running: returns task_id; client polls for completion."""
    ...

# FRAMEWORK_VERSION sourced from contract.yaml (R13 single-surface derivation)
def _load_framework_version() -> str:
    try:
        import yaml; from pathlib import Path
        data = yaml.safe_load(Path(os.getenv("CONTRACT_PATH", "/app/contract.yaml")).read_text())
        fw = data.get("framework_version")
        if fw: return f"Bible-{fw}" if not fw.startswith("Bible-") else fw
    except Exception: pass
    return "Bible-v19.1.5"
FRAMEWORK_VERSION = _load_framework_version()

# Entry point — Law 6 stateless HTTP
mcp.run(transport="streamable-http", host="0.0.0.0", port=8000, stateless_http=True)
```

## Skill Frontmatter (v19.1.5 schema)

```yaml
---
name: <pair-slug>
description: "Use for any <pair> task — <action-class-1>, <action-class-2>, ..."
version: <SERVER_VERSION>.0

# Pairing Contract binding (Law 8)
mcp_server: <pair>-mcp
mcp_tool_prefix: mcp__<pair>-mcp__
mcp_tool_count: <exact count; verified by drift-check>
mcp_auth: platform-managed | jwt | oauth | client-credentials | stdio
mcp_endpoint: https://<pair>-mcp.arjtech.in/mcp
pairing_contract_version: 1.0
pairing_contract_path: https://github.com/<ORG>/<PAIR-REPO>/blob/main/docs/pairing-contract.md
---
```

## Contract.yaml (pair repo)

Pair-specific; lives at `<pair-repo>/contract.yaml` (NOT in Bible). Consumed by Contract Enforcer at startup.

```yaml
contract_version: "1.0"
pair_slug: <slug>
pair_status: SIGNED
framework_version: "v19.1.5"     # R13 single source — tagging.py reads this
deployment_tier: 1               # R16 tier gating

# § 2 — Tool Inventory Sync
tool_inventory:
  current_count: <live>
  target_count: <post-sync>

# § 3 — Banned-Tool List (tool-name level)
banned_tools:
  - name: <deprecated_tool>
    reason: "<upstream sunset date + reason>"
    migration: "<replacement tool>"
    ban_since_version: "<server-version>"

# § 3 — Banned config patterns (Pydantic + Enforcer rejection; -32006)
banned_config_patterns:
  - tool: create_connector
    match: {service: <deprecated_service>, config_has_any_of: [<param>]}
    reason: "<deprecated-since date>"
    migration: "<new-shape>"

# § 5 — Core 8 always enabled; Domain-Conditional 2 per worksheet
guardrail_timeout: {enabled: true, default_seconds: 600, per_category: {...}}
guardrail_rate_limit: {enabled: true, backend: in-memory, per_category: {...}}
guardrail_result_cap: {enabled: true, max_rows: 1000, max_bytes: 5242880}
# guardrail_cost_preflight: REMOVED v19.1.2 — § 6 empty
guardrail_role: {enabled: false, rationale: "<why N/A for this pair>"}
guardrail_idempotency: {enabled: false, rationale: "<why N/A>"}

# § 7 — Auth
auth:
  pattern: 2-platform-managed
  session_id_binding: "<user_id>:<session_id>"

# Check 12 exemptions (time-bound — no indefinite deferrals)
spec_compliance_exemptions:
  - check: 12.3
    tool: <long_running_tool>
    rationale: "<why>"
    expires: <version>
```

## Validation (quick-check)

```bash
# Ten Laws quick check
grep -rn "__new__" src/ >/dev/null && \                               # Law 1
grep -rn "run_in_executor" src/ | wc -l | grep -q "^0$" && \          # Law 3
grep -rnE "\\*\\*kwargs" src/tools/ | wc -l | grep -q "^0$" && \      # Law 5
grep -rn "stateless_http=True" src/ >/dev/null && \                   # Law 6
grep -rn "OAuthProvider" src/ | wc -l | grep -q "^0$" && \            # Law 7 pattern-2
grep -rn 'on_duplicate="error"' src/ >/dev/null && \                  # fail-fast
echo "Ten Laws: PASS"

# v19.1.5 additions
test -f <pair-repo>/docs/pairing-contract.md && echo "Contract: in pair repo"  # Law 8
ls tests/guardrails/test_g*.py | wc -l | grep -qE "^(10|11)$" && \             # Law 9 — 10 test files (G2 gone)
  echo "Guardrails: tests present"

# Check 12 — MCP 2025-11-25 spec-compliance
curl -fsS -X POST https://<pair>-mcp.arjtech.in/mcp -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}' | \
  jq '.result.tools | map(.outputSchema != null) | all' # 12.1 — all have outputSchema
grep -E 'elicitation.*\.register\(mcp\)' src/server.py                  # 12.4

# Two-scale cascade (R13)
EXPECTED_SERVER=v1.0.0; EXPECTED_FRAMEWORK=v19.1.5; PAIR=<slug>
bash /root/aj-workspace/scripts/cascade-check.sh $EXPECTED_SERVER $EXPECTED_FRAMEWORK $PAIR

# Post-deploy drift (event-driven)
DRY_RUN=1 /root/aj-workspace/scripts/drift-check.sh --pair=<slug>
```

## Bible Source Reference (v19.1.5 — flat paths on `main`)

All paths relative to Bible repo root, branch `main`:

| Need | File |
|---|---|
| Philosophy / principles / anti-patterns | `00_PHILOSOPHY.md` |
| Ten Laws full treatment | `01_INVIOLABLE_LAWS.md` |
| 4-Tier v3.1 + Triad Trigger + trust zones | `02_ARCHITECTURE.md` |
| Server standard + MCP Primitives Registration (§ 6) + Contract Enforcer | `03_MCP_SERVER_STANDARD.md` |
| Skill standard + Layer 2 autoloader + event-driven drift | `04_SKILL_STANDARD.md` |
| Tool design — atomicity/naming/schemas/errors + § 4 (MCP 2025-11-25 MUSTs) | `05_TOOL_STANDARD.md` |
| Stack pins + supply-chain tier gating (R16) + upgrade triggers | `06_STACK_MANIFEST.md` |
| 12-section Pairing Contract + 10-guardrail catalog + Core/Domain-Conditional split | `07_PAIRING_CONTRACT.md` |
| 9-phase Upgrade Playbook + fleet queue | `08_UPGRADE_PLAYBOOK.md` |
| 12-check Completion Gate + 7-sweep Pristine Sweep + waiver policy | `09_VERIFICATION_PROTOCOL.md` |
| Domain canon + coverage audit + senior review rubric | `10_DOMAIN_COVERAGE_STANDARD.md` |
| Paired-vs-unpaired evaluation harness | `11_EVALUATION_HARNESS.md` |
| AutoResearch executable binding | `12_AUTORESEARCH_BINDING.md` |
| Upstream capability sync protocol | `13_UPSTREAM_SYNC_PROTOCOL.md` |
| Blank skill template | `templates/skill-template/SKILL.md` |
| Blank Contract template | `templates/pairing-contract-template.md` |

**Pair-specific artifacts — NOT in Bible, always in pair's own repo**:
- `<pair-repo>/docs/pairing-contract.md` (SIGNED Pairing Contract)
- `<pair-repo>/docs/pilot-learnings.md` (pilot journal)
- `<pair-repo>/docs/upstream-sync-<date>.md` (Phase 1.5 proposals)
- `<pair-repo>/contract.yaml` (machine-consumable contract)

**Reference pair**: Fivetran @ [`ARJ999/FiveTran-God-Agent-MCP`](https://github.com/ARJ999/FiveTran-God-Agent-MCP) — SIGNED, 175 tools, production at `fivetran-mcp.arjtech.in`.

## Common Mistakes (v19.1.5)

| Mistake | Fix |
|---|---|
| Using `run_in_executor` | Replace with `asyncio.to_thread(fn, *args)` (Law 3 positional) |
| `_instances = {}` dict singleton | Use `__new__` + `_instance = None` (Law 1) |
| `**kwargs` in tool handlers | Explicit Pydantic BaseModel per tool (Law 5 + G11) |
| Adding `OAuthProvider` to platform-managed server | Causes 401 on ALL requests — use `auth=None` (Law 7 Pattern 2) |
| Missing `stateless_http=True` | Add to `mcp.run()` (Law 6) |
| `on_duplicate_tools="error"` | Renamed to `on_duplicate="error"` in FastMCP 3.0 |
| **Skipping Phase 1.5 Upstream Sync** | Non-skippable. Without it, pair ships capability-stale |
| **Documenting must-adopt without implementing** | Must-adopt = shipped in this upgrade OR time-bound waiver. "Noted" is banned |
| Deploying MCP without paired skill | Law 8 violation — every production MCP has a paired skill |
| Disabling guardrail without Contract § 5 rationale | Law 9 violation — `enabled: false` requires rationale + since-version |
| `:latest` on any image tag default | Law 12 — pin to digest or specific version |
| Putting pair-specific artifacts in Bible repo | Banned. Contract + pilot-learnings + sync-proposals live in pair repo, always |
| Conflating server version with framework version | R13 — image tag tracks SERVER; `.framework` label tracks framework |
| Bumping image tag because Bible bumped | Banned — framework bumps are independent of server bumps |
| Installing nightly drift-check cron on stable pair | R14 — drift is event-driven (post-deploy + post-skill-add) |
| Redis-py dep without Triad Trigger | R7 — schrödinger dep; remove dep + module |
| Keeping `v19.0/` subdir or version-numbered folders in Bible | Bible is flat at root on `main` — no nested version folders |
| `pairing-contracts/` folder in Bible | Banned — pair contracts live in pair repos |
| Missing `outputSchema=` on data-returning tool | Check 12.1 violation — Pydantic output model MUST. For pairs >50 tools use verb-prefix classifier (see Check 12 Implementation Playbook § Pattern 1) |
| Plain `fastmcp>=3.0.2` without `[tasks]` extra when using `task=True` | Server crashes at startup with `ImportError: FastMCP background tasks require the 'tasks' extra`. Use `fastmcp[tasks]>=3.0.2` |
| Instantiating MCP primitive class without `.register(mcp)` or any usage | Aspirational dead code (R7/R11 extension). Either register OR delete the file — no in-between |
| Hand-authoring N Pydantic output models for N tools (N>50) | Unshippable. Use canonical-envelope + verb-prefix classifier pattern |
| Missing `task=True` on long-running tool (>30s) | Check 12.3 violation — agent timeouts + G4 contract break |
| Elicitation class defined but `.register(mcp)` not called | Check 12.4 violation — class exists, agent can't invoke |
| Deploying Fivetran as "first pilot" — pilot is DONE | Fivetran is the reference example. Next pair is Snowflake or Salesforce |
| Cost-preflight (G2) implementation attempt | G2 REMOVED v19.1.2. Cost attribution at warehouse layer only. `-32007` RESERVED |
| Copying v18.1 skill structure | Use `templates/skill-template/SKILL.md` — 9 sections + frontmatter with pairing fields |

## Workflow Integration

- **Pristine Sweep**: 7 categories per Bible `09 § 3`, post-upgrade (Sweep 7 is R13 cascade drift — two-scale)
- **Completion Gate**: 12 checks in Bible `09 § 1` — ALL must pass (or N/A with rationale) for Contract to SIGN
- **Decision journal**: Every upgrade records `mcp__sequential-thinking__record_decision` with `tags: mcp-upgrade,<pair>,<outcome>,st-derived` per Tier-1 Directive 5
- **Vault writeback**: Completion → `vault_submit_knowledge` with pair slug + framework version
- **Session-learnings**: Post-upgrade append `[UPGRADE] <pair> Bible-v19.1.5 server-v<X>` entry with non-obvious learnings

## Session Operating Rules — zero lapses

1. **Bible first** — read needed Bible files BEFORE touching pair code. Bible is authoritative; code is compliance.
2. **Pair-repo discipline** — all pair-specific artifacts in pair repo. Never pollute the Bible.
3. **Two-scale version clarity** — always state which scale you're bumping. Never conflate.
4. **Zero-deferred-must-adopt** — sync recommendations get implemented this upgrade or waived with expiry.
5. **Cascade-check post-bump** — run `cascade-check.sh` after any version edit. 7 surfaces must align.
6. **Gate before SIGN** — Completion Gate 12/12 GREEN is the only path to `pair_status: SIGNED`.
7. **Tier-appropriate, not tier-maximum** — R16 banned gold-plating (Tier 3 requirements on Tier 1 pair).
8. **Event-driven drift only** — no nightly cron for stable production pairs.
9. **Post-cutover Phase 10** — quality audit within 48h. Findings → `<pair-repo>/docs/pilot-learnings.md` + framework refinements if they generalize.
10. **Never claim done without running the Gate** — cheerleading is a drift class. Every `PASS` needs a command output.
