---
name: mcp-server-development
description: "Use when developing a new MCP server from scratch, upgrading an existing MCP server, performing a quality audit, or pairing a skill with its MCP. This skill is the ORCHESTRATION LAYER — operational playbook, decision flow, smart references — that consults the canonical Bible repo (https://github.com/ARJ999/MCP-God-Agent-Development-Bible) for every rule, refinement, and detail. Zero content duplication: rules live in the Bible; this skill tells you when, in what order, and where to look. Bible v19.1.12 canonical (R25-R50, 18-check Completion Gate). Reference pilots: Fivetran (SIGNED, production), tableau-mcp v6.0.1 (R45 pilot), salesforce-mcp v7.1.1 (R48 + R50 pilots), snowflake-mcp v22.0.5 (R49 pilot). Pair-specific artifacts live in each pair's own repo under docs/, not in the Bible."
version: 4.2.0
---

<!-- Changelog (skill-resident, orchestration evolution only):
v4.2.0 (2026-05-16) — Bible v19.1.12 canonization sync. AJ zero-ambiguity directive collapsed
                     v19.1.11 DRAFT + v19.1.12 DRAFT into single canonical v19.1.12 on Bible main
                     (R49 + R50 + Check 16/17/18). Updated Framework version field, Refinements Map
                     (added row 19 + marked rows 18/19 CANONIZED), Smart Reference Index, pair table
                     (salesforce v7.1.0 → v7.1.1 + R50 pilot attribution; snowflake R49 DRAFT → CANONIZED),
                     Phase 9 references (17-check → 18-check), STEP 4.7 R45/R49 selector wording (DRAFT → CANONIZED).
v4.1.0 (2026-05-06) — 57% refactor: 1616→694 lines. Removed ~1100 lines of Bible-duplicated content
                     (R38 SERF spec, Ten Laws table, Law 7/11, R32/R33, Pairing Contract detail,
                     Guardrails detail, Check 12 detail, R7/R13/R14/R16, Gold Stack, 4-Tier,
                     10-Phase Methodology, 9-Phase Playbook, Pristine Sweep detail, Upstream Sync
                     detail, Check 12 Implementation Patterns 1-6, FastMCP Critical Patterns,
                     Skill Frontmatter spec, Contract.yaml schema, Common Mistakes full table —
                     all replaced with smart references). Added Bible↔Skill Sync Protocol +
                     What Goes Where Litmus Test (decision tree + worked examples + ongoing
                     discipline rules). Added Phase 4 STEP 4.7 (R45/R49 architecture-aware
                     static-arity audit) + Phase 9 STEP 9.3 reference to 17-check post-renumber
                     gate + Phase 10 STEP 10.5 (Bible upstream cascade). Updated all pair-table
                     rows to current production state.
v4.0.0 (2026-05-06 earlier) — partial sync to Bible v19.1.10 + v19.1.11 DRAFT (Bible↔Skill sync
                              protocol added; some duplication remained).
v3.2.3 (pre-2026-05-06) — Bible v19.1.7-era. 1616 lines with substantial Bible duplication. -->


# MCP Server Development — Orchestration Layer for the MCP God Agent Development Bible

> **Architecture**: this skill = orchestration + smart references + operational playbook. The **Bible repo** = canonical content (laws, refinements, full specs, templates). **Zero content duplication.** Skill and Bible are paired; they always stay synchronized.
>
> ⚠️ **ABSOLUTE PROTOCOL — PULL CANONICAL BIBLE FIRST (no exceptions)**
>
> Before referencing any Bible version, law (Lxx), refinement (Rxx), check (Cxx), or chapter §:
> ```bash
> cd /root/aj-workspace/MCP-God-Agent-Development-Bible && git pull --ff-only origin main
> grep -m1 'Latest refinement' README.md
> ```
> Container labels (`com.<pair>.bible`, `BIBLE_VERSION`, `contract.yaml.framework_version`, `/health.framework`) reflect build-time and may be 1+ minor versions stale. **Bible main is the only source of truth.** Working from a stale local clone violates this protocol.
>
> ⚠️ **NEVER conduct external research for framework decisions.** This skill + the Bible are the COMPLETE operational reference. WebSearch / WebFetch on FastMCP, MCP SDK, MCP protocol, or framework patterns is BANNED. External research is permitted ONLY for (a) platform vendor API documentation during Phase 1.5 Upstream Capability Sync, and (b) authoritative upstream release feeds during stack currency scans.

## Bible Repository — Source of Truth

| | |
|---|---|
| **Canonical repo** | [ARJ999/MCP-God-Agent-Development-Bible](https://github.com/ARJ999/MCP-God-Agent-Development-Bible) |
| **Authoritative branch** | `main` (flat repo layout) |
| **Local clone** | `/root/aj-workspace/MCP-God-Agent-Development-Bible/` — `git pull --ff-only origin main` before every consultation |
| **Framework version** | **v19.1.12** canonized 2026-05-16 (R49 intra-class arity audit + R50 per-tool timeout enforcement + Checks 16/17/18; cumulative R25–R50, 18-check Completion Gate) |
| **MCP Spec** | 2025-11-25 |
| **Python SDK** | `mcp >= 1.27.0, < 2` (FastMCP 3.0+ bundled) |
| **Reference pilots** | Fivetran (SIGNED), tableau-mcp v6.0.1 (R45), salesforce-mcp v7.1.1 (R48 + R50), snowflake-mcp v22.0.5 (R49) |
| **Pair artifacts** | NEVER in Bible — always `<pair-repo>/docs/pairing-contract.md`, `<pair-repo>/docs/pilot-learnings.md`, `<pair-repo>/docs/upstream-sync-<date>.md` |

## Bible ↔ Skill Sync Protocol — paired, never drift

The Bible and this skill are a paired unit. They stay in lockstep through these rules:

| Trigger | Required action |
|---|---|
| Bible canonizes a minor version (e.g., v19.1.10 → v19.1.11) | This skill's **Framework version** field + **Refinements Map** + **Smart Reference Index** updated within 24h or before next pair-upgrade work, whichever first |
| Bible adds a new refinement (Rxx) | Add row to Refinements Map; add row to Smart Reference Index; do NOT restate the rule's content here |
| Bible adds a new check (Cxx) | Update Operational Playbook's Phase 9 reference if the check fires there; do NOT restate the check's content here |
| Skill changes its playbook structure | No Bible change required — orchestration is skill-resident |
| Pair upgrades to a newer Bible version | Skill's pair table row + Refinements Map "pilot" attribution updated same op |
| `/health.framework` of any deployed pair lags Bible main | That pair is owed a Bible-sync minor patch — flag in Phase 1 of next upgrade |

**Drift detection**: at session start, agent compares this skill's "Framework version" vs Bible README header line 1. Mismatch → run sync pass on this skill before doing pair work.

**Anti-pattern**: copying Bible content into this skill. If a section feels duplicative of Bible material, replace it with a pointer (`see Bible §N` or `Bible chapter file_X.md § Y`). Skill content = orchestration + decision logic + smart references. Bible content = rules + refinements + specs.

## What Goes Where — Litmus Test (apply on EVERY change, ongoing)

Every time anything is added, modified, or refined, run it through this test BEFORE writing it anywhere:

### Bible-resident content (canonical, cited, framework-wide)

A piece of content goes in the Bible if it is:

- **A rule, law, refinement, or check** with a stable identifier (Lxx, Rxx, Cxx, §N) that gets CITED elsewhere
- **A specification** that defines WHAT MUST be true (schema, contract shape, MCP wire-format requirement)
- **A template** that pairs copy from (`templates/skill-template/SKILL.md`, `templates/pairing-contract-template.md`)
- **A reference script** that pairs invoke as-is (`framework/scripts/wrapper_arity_audit.py`, `intra_class_arity_audit.py`, `symmetric_reader_audit.py`)
- **A cross-pair invariant** — applies to every MCP server identically regardless of platform
- **An anti-pattern catalog** — what is BANNED and why (R29 hallucinated SQL, Pattern 2 + OAuthProvider, etc.)
- **An architecture description** — 4-Tier, MCP Primitives, Triad Trigger conditions, version-scale model
- **A stack manifest** — Gold Stack pins, supply-chain tier requirements
- **A detailed phase specification** — the WHAT and WHY of an upgrade phase (the HOW + WHEN is skill-resident)
- **A retrospective learning that generalizes** — pattern surfaced in pair X, applies to pairs Y/Z (R45 from tableau-mcp, R49 from snowflake-mcp)

### Skill-resident content (orchestration, decision, live state)

A piece of content goes in the skill if it is:

- **WHEN to invoke** — trigger conditions, decision flow, branching logic
- **WHICH Bible chapter to consult for what** — the routing table (Bible Chapter Quick-Map)
- **ORDER of operations** — Phase 0 → 0.5 → 1.5 → 2 → 3 → 4 → ... — the sequence is orchestration
- **HOW to apply a Bible rule operationally** — the runbook step that consults the rule, not the rule itself
- **LIVE STATE** — current pair table, current canonical Bible version, current production pilots, current `/health` shapes
- **SMART REFERENCES** — every CONSULT pointer, every "see Bible §X for Y" line
- **OPERATIONAL DISCIPLINE** — session operating rules about HOW we work (not WHAT the framework requires)
- **DRIFT DETECTION** — this skill ↔ Bible sync protocol, version-mismatch triage
- **SYMPTOM-TO-CHAPTER TRIAGE** — "if you see error X, look in Bible chapter Y"
- **META-DISCIPLINE** — like this section itself: how to evolve the system, not what the system requires

### The decision tree (run on every change)

```
New piece of content arrives
        │
        ▼
Is it a RULE that applies to every MCP server identically? ────► Bible
        │ no
        ▼
Does it have a STABLE IDENTIFIER (Lxx/Rxx/Cxx) that other content cites? ────► Bible
        │ no
        ▼
Is it a TEMPLATE / SCRIPT / SPECIFICATION pairs consume directly? ────► Bible
        │ no
        ▼
Is it WHEN/WHERE/HOW to consult a Bible rule? ────► Skill (with CONSULT pointer)
        │ no
        ▼
Is it LIVE STATE (current versions, current pairs, current pilots)? ────► Skill
        │ no
        ▼
Is it ORCHESTRATION DISCIPLINE about how the skill itself operates? ────► Skill
        │ no
        ▼
ELSE: re-examine — content that fits no bucket may not need to exist
```

### Worked examples (recent decisions)

| Content | Where it goes | Why |
|---|---|---|
| R49 spec — intra-class AST audit rule | Bible `18_*.md` | Stable identifier (R49), framework-wide rule, applies to every all-in-one-class server |
| `intra_class_arity_audit.py` script | Bible `framework/scripts/` | Reference script pairs consume directly |
| "Run R49 audit at Phase 9 Check 17 if all-in-one-class" | Skill (Operational Playbook STEP 4.7 + STEP 9.3) | Orchestration — when/how to apply R49 |
| "snowflake-mcp v22.0.5 = R49 pilot" | Skill (pair table + Refinements Map) | Live state — current pilot attribution |
| Bible-pull protocol ("git pull before any consultation") | Skill (prologue) | Orchestration discipline about how this skill itself operates |
| Codex hard-gate degrade-signature list | Hook source code (`/root/.claude/hooks/codex-pre-pr-gate.py`) | Neither Bible nor skill — it's harness operational config; lives in workspace `.claude/hooks/` |
| "Two-layer vs all-in-one-class architecture classification" | Bible `18_*.md § Migration Path` | The classification rubric is a framework-wide rule |
| "If pair is two-layer, run R45 audit; if all-in-one-class, run R49" | Skill (Operational Playbook STEP 4.7 routing) | Orchestration — which audit applies when |
| Verb-prefix output_schema classifier code | Bible `03_MCP_SERVER_STANDARD.md § Pattern 1` | Reference implementation pattern, cross-pair |
| "Fivetran v19.1.1 still on Bible v19.1.7 — Bible-sync owed" | Skill (pair table) | Live state + drift signal |

### The ongoing discipline (god-grade evolution)

Every session that touches the framework:

1. **At session start**: pull canonical Bible main (Tier 1 Directive 7). Compare this skill's framework-version field vs Bible README header. Mismatch → schedule sync pass.
2. **When adding new content**: run the litmus test BEFORE writing. If unsure, default to Bible (it's the source of truth) and add a smart reference here.
3. **When a pair upgrade surfaces a new pattern**: file PR/DRAFT to Bible first, then update this skill's Refinements Map row. Phase 10 STEP 10.5 enforces.
4. **When this skill needs a section that re-states a Bible rule**: STOP. Replace with `CONSULT Bible §X` pointer. The duplication is the bug.
5. **When the Bible repo updates**: this skill's version field, Refinements Map, and Bible chapter map sync within the same session. Drift = paired-unit failure.
6. **Pristine Sweep**: every closure scans this skill for content that has migrated INTO the Bible since last sync — replace with pointer.

This discipline is itself ongoing — additions to "what goes where" should accumulate as worked examples in this section as the system evolves. Never lose the lesson; never restate it elsewhere.

## Framework Refinements Map (R25–R50 cumulative, all CANONIZED)

Pure-reference table. Rule content lives in the linked Bible files — never restated here.

| Bible file | Range | Status | Headline |
|---|---|---|---|
| [`14_FRAMEWORK_REFINEMENTS_v19.1.7.md`](https://github.com/ARJ999/MCP-God-Agent-Development-Bible/blob/main/14_FRAMEWORK_REFINEMENTS_v19.1.7.md) | R25–R37 (13 refinements) | CANONIZED 2026-04-22 | Cascade R13, drift R14, Census R32, Result-shape R33, Law 11 R29, SERF R38 |
| [`15_FRAMEWORK_REFINEMENTS_v19.1.8.md`](https://github.com/ARJ999/MCP-God-Agent-Development-Bible/blob/main/15_FRAMEWORK_REFINEMENTS_v19.1.8.md) | R41–R44 (salesforce-mcp v7.0.0 pilot) | CANONIZED | Cross-cutting feature gating (FEATURE_NOT_ENABLED unification + Pattern A platform-managed transport) |
| [`16_FRAMEWORK_REFINEMENTS_v19.1.9.md`](https://github.com/ARJ999/MCP-God-Agent-Development-Bible/blob/main/16_FRAMEWORK_REFINEMENTS_v19.1.9.md) | R45–R47 (tableau-mcp v6.0.1 pilot) | CANONIZED 2026-05-03 | **Wrapper-impl arity discipline** — R45 AST audit (two-layer pattern) + R46 kwargs-by-name + R47 impl-widening recipe |
| [`17_FRAMEWORK_REFINEMENTS_v19.1.10.md`](https://github.com/ARJ999/MCP-God-Agent-Development-Bible/blob/main/17_FRAMEWORK_REFINEMENTS_v19.1.10.md) | R48 + Check 15 (salesforce-mcp v7.1.0 pilot) | CANONIZED 2026-05-06 | **Complete-Capability Invariant** — operator-intent tools deliver in one round-trip + symmetric metadata reader/writer audit |
| [`18_FRAMEWORK_REFINEMENTS_v19.1.11.md`](https://github.com/ARJ999/MCP-God-Agent-Development-Bible/blob/main/18_FRAMEWORK_REFINEMENTS_v19.1.11.md) | R49 + Check renumber (snowflake-mcp v22.0.5 pilot) | CANONIZED 2026-05-16 | **Intra-class arity audit** — sibling to R45 for all-in-one-class servers + Check 13 collision resolution (R45 → 16, R49 → 17) |
| [`19_FRAMEWORK_REFINEMENTS_v19.1.12.md`](https://github.com/ARJ999/MCP-God-Agent-Development-Bible/blob/main/19_FRAMEWORK_REFINEMENTS_v19.1.12.md) | R50 + Check 18 (salesforce-mcp v7.1.1 pilot) | CANONIZED 2026-05-16 | **Per-tool timeout enforcement** — `asyncio.wait_for` wrap at `install_annotator` boundary with category-aware caps; resolves 665+ unbounded `asyncio.to_thread` sites |

**Architecture-aware static-arity audit selection** (R45 + R49 are mutually exclusive per pair):

| Server pattern | Audit | Detect by |
|---|---|---|
| Two-layer FastMCP (`@mcp.tool()` wrapper + separate `api_client` class) | **R45** — `wrapper_arity_audit.py` | Separate `server.py` + `core/api_client.py`; wrapper calls `<client>.<method>(...)` |
| All-in-one-class (single class file, intra-class dispatch) | **R49** — `intra_class_arity_audit.py` | One large class file; dispatch is `self.<method>(...)` |
| Single-method / fully-self-contained wrappers | Neither (rely on R39 + Codex) | No `self.<method>` or `<client>.<method>` indirection inside `@mcp.tool()` body |

R39 (realistic-args census) and Codex hard-gate (cross-model review) apply unconditionally regardless of pattern. R50 (per-tool timeout enforcement at `install_annotator`) applies unconditionally to every FastMCP-based pair regardless of architecture.

**Migration path for pairs still on v19.1.6 or earlier**: adopt R25–R50 in Bible-version order. Full R25–R50 adoption expected for any major-version upgrade post 2026-05-16. Pair-side `phase9_completion_gate.sh` templates need mechanical Check renumber (R45 Check 13 → 16, add Check 17 if all-in-one-class, add Check 18 for all FastMCP pairs) on next refresh.

## What is a "Pair"? — Definitive Glossary (read this BEFORE the playbook)

A **pair** = one MCP server + one paired skill, governed by ONE Pairing Contract. They ship together, version together, get upgraded together. Neither half exists in production without the other.

```
                    ┌───────────────── PAIR ─────────────────┐
                    │                                         │
   /opt/mcp-servers/<slug>/   ←─ Pairing Contract ─→  /root/.claude/skills/<slug>/
   (the MCP server,                                      (the skill,
    Docker container,                                     SKILL.md + references/,
    contract.yaml,                                        domain expertise,
    src/, scripts/, tests/)                               gotcha registry)
                    │                                         │
                    └─ together = ONE pair = SIGNED unit ─────┘
```

**Production pairs on this VPS** (state at 2026-05-06; salesforce row refreshed 2026-06-06 → v7.1.4/684):

| Pair slug | MCP server | Paired skill | Server v | Bible fw | Static-arity audit |
|---|---|---|---|---|---|
| `snowflake` | snowflake-mcp.arjtech.in (~880 tools) | `/root/.claude/skills/snowflake/` | v22.0.5 | v19.1.10 (Bible-sync owed → v19.1.12) | R49 — pilot |
| `fivetran` | fivetran-mcp.arjtech.in (175 tools) | `/root/.claude/skills/fivetran/` | v19.1.1 | v19.1.7 (Bible-sync owed → v19.1.12) | TBD per pattern; Bible-sync owed |
| `salesforce` | salesforce-mcp.arjtech.in (684 tools) | `/root/.claude/skills/salesforce/` | v7.1.4 | v19.1.10 (Bible-sync owed → v19.1.12; R50 runtime already live) | R45 (R48 + R50 pilots) |
| `tableau` | tableau-mcp.arjtech.in (521 tools) | `/root/.claude/skills/tableau/` | v6.0.1 | v19.1.9 (Bible-sync owed → v19.1.12) | R45 — pilot |
| `blue` | blue-mcp.arjtech.in (129 tools) | `/root/.claude/skills/blue/` | v4.0.0 | v19.1.3 (Bible-sync owed → v19.1.12) | TBD; Bible-sync owed |
| `aws` | aws-mcp.arjtech.in (~3,006 tools) | `/root/.claude/skills/aws/` | v4.0.0 | v19.1.10 (Bible-sync owed → v19.1.12) | Registry-dispatch pattern (R50 runtime applies via install_annotator wrap; sibling registry-dispatch pattern formalization deferred to v19.1.13 candidate per PR #25) |

### Two distinct invocation modes — DO NOT CONFUSE

| When the user asks for... | Invoke this skill | Why |
|---|---|---|
| Domain work using a specific platform (e.g., "query Snowflake", "list Fivetran connectors") | The **pair skill** (e.g., `snowflake`, `fivetran`) | Pair skill carries domain expertise + tool inventory + gotchas |
| Build, upgrade, audit, migrate, or modify a pair's MCP+skill itself | **`mcp-server-development`** (this skill) | This skill carries the orchestration playbook; it consults the Bible for rules |

**Concrete example**: AJ asks "upgrade snowflake to Bible v19.2" → invoke `mcp-server-development` (the upgrade playbook), which consults Bible chapters in sequence, AND coordinates the upgrade of BOTH the snowflake MCP server (`/opt/mcp-servers/snowflake-mcp/`) AND the snowflake skill (`/root/.claude/skills/snowflake/`) IN LOCKSTEP. The pair stays as a unit.

### Pair invariants (never violated, ever)

1. **Lockstep version** — pair MCP server version + pair skill version + Pairing Contract `tool_inventory.target_count` must all match the live `tools/list` count at all times. Drift = Phase 5 + R32 cascade-check failure.
2. **Single source of truth** — Pairing Contract lives in pair-repo `<pair-repo>/docs/pairing-contract.md`. Skill `pairing_contract_path` frontmatter points to it. Bible NEVER carries pair-specific contract content.
3. **Skill mirror** — `/root/.claude/skills/<slug>/SKILL.md` and `/home/claude/.claude/skills/<slug>/SKILL.md` are byte-identical (R11 dual-mirror). On this VPS they're symlinked/hardlinked, so this is automatic.
4. **R11 cleanup** — pair MCP repo MUST NOT contain `skill/` subdirectory. Skill content lives ONLY in the global skills dir.
5. **Pair upgrade is atomic** — never bump server version without bumping skill `mcp_tool_count` in same cycle. Cascade-check (R13) verifies surfaces align across both version scales.

### When a NEW pair is being built (not upgrade — net-new MCP)

Same skill (`mcp-server-development`), but the playbook follows the new-server path (see Decision Flow below). Phase 0 includes pair-slug selection, GitHub repo creation, and skill scaffolding via `templates/skill-template/SKILL.md` from the Bible.

## Bible Chapter Quick-Map (smart references — single source of routing)

When the Operational Playbook says `CONSULT § <number>` or `CONSULT § <name>`, the chapter is in `ARJ999/MCP-God-Agent-Development-Bible` repo, file `0N_<TOPIC>.md`. This is the only routing table — never duplicated, never restated.

| Bible chapter | Skill playbook step that uses it |
|---|---|
| `00_PHILOSOPHY.md` | Onboarding — read once for principles |
| `01_INVIOLABLE_LAWS.md` | Phase 4 (every law check); contains Ten Inviolable Laws + Law 7 5-pattern auth + Law 11 R29 Upstream Verification |
| `02_ARCHITECTURE.md` | Phase 2/3 (4-Tier v3.1 + R7 Triad Trigger + R13 cascade discipline) |
| `03_MCP_SERVER_STANDARD.md` | Phase 4.1–4.3 (server.py + install_annotator + Contract Enforcer + MCP Primitives Registration § 6 + Check 12 Implementation Patterns) |
| `04_SKILL_STANDARD.md` | Phase 5 (skill upgrade — frontmatter, 9 sections, dual-mirror) |
| `05_TOOL_STANDARD.md` | Phase 4.4 (tool design + outputSchema + ToolAnnotations + Task R24 + R29 + R38 SERF) |
| `06_STACK_MANIFEST.md` | Phase 3 (Gold Stack pin bumps + Supply-Chain Tier Gating R16) |
| `07_PAIRING_CONTRACT.md` | Phase 2 (12-section contract.yaml + Core 8 + Domain-Conditional 2 guardrails) |
| `08_UPGRADE_PLAYBOOK.md` | Phase 1, 9 — full 9-Phase Bible-side upgrade detail (companion to this skill's Operational Playbook) |
| `09_VERIFICATION_PROTOCOL.md` | Phase 0.5, 9 — Census + 12-Check Completion Gate + 7-category Pristine Sweep |
| `10_DOMAIN_COVERAGE_STANDARD.md` | Phase 1.5 (capability gap analysis) |
| `11_EVALUATION_HARNESS.md` | Phase 8 (paired-vs-unpaired evaluation, Tier 2+) |
| `12_AUTORESEARCH_BINDING.md` | Phase 5 (skill optimization loops) |
| `13_UPSTREAM_SYNC_PROTOCOL.md` | Phase 1.5 mandatory 6-step sync |
| `14_FRAMEWORK_REFINEMENTS_v19.1.7.md` | Throughout — R25–R37 (cascade R13, drift R14, Census R32, Result-shape R33, Law 11 R29, SERF R38) |
| `15_FRAMEWORK_REFINEMENTS_v19.1.8.md` | Throughout — R41–R44 (FEATURE_NOT_ENABLED unification, Pattern A transport) |
| `16_FRAMEWORK_REFINEMENTS_v19.1.9.md` | Phase 4 + Phase 9 Check 16 — R45–R47 (wrapper-impl arity, two-layer) |
| `17_FRAMEWORK_REFINEMENTS_v19.1.10.md` | Phase 4 + Phase 9 Check 15 — R48 (Complete-Capability Invariant) |
| `18_FRAMEWORK_REFINEMENTS_v19.1.11.md` | Phase 4 + Phase 9 Check 17 — R49 (intra-class arity, all-in-one-class) |
| `19_FRAMEWORK_REFINEMENTS_v19.1.12.md` | Phase 4 + Phase 9 Check 18 — R50 (per-tool timeout enforcement at install_annotator) |
| `templates/pairing-contract-template.md` | Phase 2 (blank Contract template — copy + fill) |
| `templates/skill-template/SKILL.md` | Phase 5 (blank skill scaffold for new pairs) |
| `framework/scripts/wrapper_arity_audit.py` | Phase 9 Check 16 (R45 enforcement, two-layer servers) |
| `framework/scripts/intra_class_arity_audit.py` | Phase 9 Check 17 (R49 enforcement, all-in-one-class servers) |
| `framework/scripts/wait_for_audit.py` (authoring deferred to v19.1.13 candidate) | Phase 9 Check 18 (R50 enforcement; runtime guard is live in salesforce-mcp v7.1.1) |
| `framework/scripts/symmetric_reader_audit.py` | Phase 9 Check 15 (R48 enforcement) |

### How to fetch Bible content (three options, in order of preference)

1. `gh repo view ARJ999/MCP-God-Agent-Development-Bible --json defaultBranchRef` to confirm latest sha
2. `gh api repos/ARJ999/MCP-God-Agent-Development-Bible/contents/<FILE>.md --jq .content | base64 -d` to read a chapter without cloning
3. `cd /root/aj-workspace/MCP-God-Agent-Development-Bible && git pull --ff-only origin main` then read locally (preferred for multi-chapter consultation; this is the local clone path enforced by the Bible-pull discipline)

### How learnings cascade

- **Skill ← Bible**: when Bible bumps, this skill's Refinements Map + Bible chapter map + version field update within the same session.
- **Bible ← Pair**: every pair upgrade that surfaces a new pattern (R<N>) MUST author a PR or DRAFT entry to the Bible repo. Phase 10 closure includes this. Recent surfaces: tableau-mcp 2026-05-03 → R45–R47 (v19.1.9), salesforce-mcp 2026-05-06 → R48 + Check 15 (v19.1.10), snowflake-mcp 2026-05-06 → R49 (v19.1.11), salesforce-mcp 2026-05-11 → R50 + Check 18 (v19.1.12).

## Two Independent Version Scales (R13)

Every deployed pair reports BOTH scales — never conflate them. Detail in Bible `02_ARCHITECTURE.md` + `14_*.md § R13`.

| Scale | What it tracks | Where to look |
|---|---|---|
| **Server version** | Pair's own app version (e.g., `19.1.1`, `v22.0.5`) | image tag, `com.<pair>.version` label, `/health.version`, Dockerfile ENV |
| **Framework version** | Bible documentation version (e.g., `Bible v19.1.12`) | `com.<pair>.bible` label, `contract.yaml.framework_version`, `/health.framework` |

Bumping one scale does NOT force bumping the other — they move independently. Cascade-check verifies both.

## Decision Flow

```
┌─────────────────────────────────┐
│ MCP + skill task                │
└────────────────┬────────────────┘
                 ▼
      ┌──────────────────────┐
      │ New / Upgrade / Audit │
      └──────────────────────┘
         │        │        │
         │        │        └── quality audit → 12-Check Completion Gate (Bible §09)
         │        │                            + Pristine Sweep (Bible §09 §3)
         │        │
         │        └── existing → Operational Playbook below (consult Bible §08)
         │                      MANDATORY Phase 1.5 Upstream Capability Sync
         │
         └── new → Operational Playbook (Phase 0 = scaffold; consult Bible §03 §9)
                   Pairing Contract authored concurrently in <pair-repo>/docs/
```

## Operational Playbook — STEP-BY-STEP RUNBOOK (the skill's value-add)

> **Read this FIRST. Every upgrade follows this exact sequence. Each step has a HARD GATE — do NOT proceed to the next step until the gate passes. Every CONSULT line is a smart reference to a Bible chapter — the rules live there, not here.**

### How to use this runbook

For each phase below:
- **STEPS** are numbered actions in mandatory order
- **CONSULT** lines tell you which Bible chapter or refinement to read for the rules (rules live in Bible, not duplicated here)
- **VERIFY** lines tell you the exact command/probe to run
- **GATE** lines define the pass criterion — proceed ONLY if true
- **STOP-IF-FAIL** lines tell you what to do when a gate fails (never silently work around)

If a step says "verified live", that means a curl/probe against the live system — NOT "I think it works" or "the docs say so" alone.

---

### PHASE 0 — Pre-Flight (MANDATORY before any upgrade work)

**STEP 0.1** — Identify the pair and current state
- Action: `cd /opt/mcp-servers/<pair-slug> && cat pyproject.toml | grep version`
- Action: `docker ps --filter name=<pair-slug> --format '{{.Image}} {{.Status}}'`
- VERIFY: pair exists, container is healthy, current version captured
- GATE: pair is operational pre-upgrade
- STOP-IF-FAIL: file an issue, do not proceed

**STEP 0.2** — Capture .env baseline (`.env` is sacrosanct — non-negotiable)
- CONSULT: Bible `08_UPGRADE_PLAYBOOK.md § Pre-flight Rule 0` (full rationale + safety clause)
- Action: `mkdir -p /tmp/<slug>-baseline-$(date +%F) && sha256sum /opt/mcp-servers/<slug>/.env > /tmp/<slug>-baseline-$(date +%F)/env-hash.txt`
- VERIFY: hash file exists with non-empty content
- GATE: env-hash.txt captured
- STOP-IF-FAIL: never proceed without baseline — rollback safety depends on this

**STEP 0.3** — Snapshot live tool surface
- Action: `curl -fsS -X POST https://<slug>-mcp.arjtech.in/mcp -H 'Content-Type: application/json' -H 'Accept: application/json, text/event-stream' -H 'MCP-Protocol-Version: 2025-11-25' -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}' | sed -n 's/^data: //p' > /tmp/<slug>-baseline-$(date +%F)/tools-list-pre.json`
- VERIFY: `jq '.result.tools | length' /tmp/<slug>-baseline-$(date +%F)/tools-list-pre.json` returns the expected baseline count
- GATE: tool inventory snapshotted
- STOP-IF-FAIL: live MCP isn't responding — fix that first

**STEP 0.4** — Tag rollback image + create backup branch
- Action: `docker tag <pair-slug>-<pair-slug>:latest <pair-slug>-<pair-slug>:v<CURRENT>-pre-upgrade`
- Action: `cd /opt/mcp-servers/<pair-slug> && git checkout -b backup/v<CURRENT>-<pair-slug>-$(date +%F) && git push -u origin backup/v<CURRENT>-<pair-slug>-$(date +%F)`
- VERIFY: `docker images <pair-slug>-<pair-slug> | grep pre-upgrade` non-empty
- GATE: rollback path exists
- STOP-IF-FAIL: never start upgrade without rollback tag

---

### PHASE 0.5 — Tool Health Census (MANDATORY)

**STEP 0.5.1** — Author the Census harness with realistic-args templates
- CONSULT: Bible `14_*.md § R32 Comprehensive Tool Validation Protocol` (4-layer probe spec)
- CONSULT: Bible `14_*.md § R29 Law 11 Upstream Verification` (banned pattern-inferred guesses)
- Action: write `scripts/tool_health_census.py` per the reference implementation in any signed pair (snowflake-mcp v22, fivetran-mcp v19 are working examples — but the harness itself is pair-agnostic)
- The harness MUST include all 4 probe layers per R32: empty-args probe, realistic-args probe with per-family templates, response-shape probe (catches R33-fixable errors), server-stability probe (rate-limit floor + post-burst /health restart-test).
- VERIFY: `python3 -m py_compile scripts/tool_health_census.py && echo OK`
- GATE: harness compiles + has all 4 probe layers per R32 spec

**STEP 0.5.2** — Run Census and triage
- Action: `python3 scripts/tool_health_census.py --rate-limit-sleep 0.05 2>&1 | tee docs/census-pre-upgrade-$(date +%F).log`
- VERIFY: census completes without exhausting server (`/health` still returns healthy after run)
- GATE: census output produces CSV with outcome classification per tool
- STOP-IF-FAIL: if server returns 404 cascade mid-census, restart server, lower rate, restart census from scratch

**STEP 0.5.3** — Classify findings → action plan
- CONSULT: Bible `14_*.md § R32` outcome rubric (PASS / BROKEN / FEATURE_NOT_ENABLED / probe-gap)
- For each `BROKEN` row: classify as TRUE_BUG (SQL syntax error, function doesn't exist) vs PROBE_GAP (template not realistic enough) vs FEATURE_NOT_ENABLED
- For each TRUE_BUG: must be fixed or retracted before SIGN
- For each PROBE_GAP: improve the probe template (cite vendor docs URL)
- For each FEATURE_NOT_ENABLED: tool is fine, document as expected
- VERIFY: every BROKEN row has assigned classification + remediation in `docs/census-triage-$(date +%F).md`
- GATE: triage doc written + AJ-reviewable
- STOP-IF-FAIL: do NOT proceed to Phase 4 with un-triaged BROKEN tools

---

### PHASE 1.5 — Upstream Capability Sync (MANDATORY)

**STEP 1.5.1** — Run the 6-step sync protocol
- CONSULT: Bible `13_UPSTREAM_SYNC_PROTOCOL.md § 2.3` (full 6-step protocol)
- Action: collect upstream inventory (REST API, SDK, platform features, MCP spec, deprecations)
- Action: snapshot current pair coverage
- Action: diff → classify must-adopt / should-adopt / optional / ignored
- Action: author `<pair-repo>/docs/upstream-sync-<DATE>.md`
- VERIFY: sync proposal committed
- GATE: must-adopt count documented; zero-deferred-must-adopt rule applies (no "noted but not done")

**STEP 1.5.2** — VERIFY EVERY MUST-ADOPT BEFORE PHASE 4 (R29)
- CONSULT: Bible `14_*.md § R29 Law 11` + R31 Per-tool VERIFY GATE
- For each must-adopt item that involves new SQL/REST/CLI calls:
  - Either: live-probe the function/endpoint exists (`SELECT function_name FROM <PLATFORM>.INFORMATION_SCHEMA.FUNCTIONS WHERE function_name = '<NAME>'`)
  - Or: cite the vendor docs URL of the function signature in the upstream-sync proposal
- VERIFY: every must-adopt has either a live-probe receipt or a vendor docs URL
- GATE: zero unverified must-adopts
- STOP-IF-FAIL: a must-adopt without verification is a R29 violation — DO NOT add it to capability_adds_*.py

---

### PHASE 2 — Pairing Contract (MANDATORY — Bible Law 8)

**STEP 2.1** — Draft the 12-section contract.yaml
- CONSULT: Bible `07_PAIRING_CONTRACT.md` (12-section spec + Core 8 + Domain-Conditional 2)
- Action: copy `templates/pairing-contract-template.md` from Bible repo, fill all 12 sections (§6 is empty per G2 removal in v19.1.2)
- VERIFY: `python3 -c "import yaml; yaml.safe_load(open('contract.yaml'))"` passes
- GATE: contract.yaml syntactically valid + all sections populated

**STEP 2.2** — Declare deployment_tier per R16
- CONSULT: Bible `06_STACK_MANIFEST.md § 1.6.1` Supply-Chain Tier Gating R16
- Action: choose Tier 1 / 2 / 3 with rationale (Tier 1 = personal VPS R&D; Tier 2 = single-org production; Tier 3 = multi-tenant enterprise)
- VERIFY: `grep '^deployment_tier:' contract.yaml` returns 1/2/3
- GATE: tier declared with rationale
- STOP-IF-FAIL: never gold-plate (Tier 3 reqs on Tier 1 pair) — banned

**STEP 2.3** — Banned-tool list for Gen 1 / deprecated tools
- CONSULT: Bible `07_PAIRING_CONTRACT.md § 3` (banned-tool fields spec)
- For each tool to be banned: name + reason + migration helper + ban_since_version
- VERIFY: `grep -A1 '^banned_tools:' contract.yaml` shows each banned tool with all 4 fields
- GATE: every banned tool has migration path documented

**STEP 2.4** — Architecture classification (R49 v19.1.11 CANONIZED — pair-contract artifact)
- CONSULT: Bible `18_FRAMEWORK_REFINEMENTS_v19.1.11.md § Migration Path` (architecture YAML block)
- Action: declare `architecture: { pattern, rationale, static_arity_audit, audit_command }` in contract.yaml
- VERIFY: pattern is one of `two-layer | all-in-one-class | self-contained` and `static_arity_audit` matches the pattern (R45 / R49 / none)
- GATE: architecture declared; pair-side `phase9_completion_gate.sh` runs the corresponding audit

---

### PHASE 3 — Stack Bump (MANDATORY pyproject + Dockerfile + compose)

**STEP 3.1** — Update pyproject pins per Bible Gold Stack
- CONSULT: Bible `06_STACK_MANIFEST.md § 1` (current pins + floors + supply-chain tier requirements)
- Action: update pyproject.toml with current Bible-mandated minimums
- VERIFY: `uv lock --upgrade-package <each>` succeeds
- GATE: lock file regenerated, all pins meet Bible floors

**STEP 3.2** — Dockerfile + docker-compose updates (R13 cascade)
- CONSULT: Bible `14_*.md § R13` Cascade-Discipline (atomic-bump surfaces across 2 version scales)
- Action: bump all surfaces atomically (image tag, labels, env vars, /health.version, /health.framework, contract.yaml framework_version, pilot-learnings, all source-default version strings)
- VERIFY: `bash /root/aj-workspace/scripts/cascade-check.sh v<NEW> Bible-v<NEW> <pair-slug>` exits 0
- GATE: all surfaces aligned across both version scales (server + framework)

**STEP 3.3** — .env hash check post-bump
- Action: `NEW_HASH=$(sha256sum .env | awk '{print $1}'); BASE=$(cat /tmp/<slug>-baseline-*/env-hash.txt | awk '{print $1}'); [[ "$NEW_HASH" == "$BASE" ]] && echo OK || exit 1`
- GATE: .env unchanged
- STOP-IF-FAIL: investigate immediately — never proceed if .env touched

---

### PHASE 4 — Server-side Enforcement + Capability Adds (BIG PHASE — most discipline required)

**STEP 4.1** — install_annotator wired BEFORE register_all_tools
- CONSULT: Bible `03_MCP_SERVER_STANDARD.md § install_annotator pattern` (with R24 backend gate + R33 result-shape wrap)
- Action: server.py `install_annotator(mcp)` MUST be called BEFORE any tool registration
- VERIFY: `grep -B2 -A2 'install_annotator(mcp)' src/<pkg>/server.py` shows it precedes `register_all_tools`
- GATE: install_annotator wraps mcp.tool before any registration call

**STEP 4.2** — install_annotator includes R33 result-shape wrapping
- CONSULT: Bible `14_*.md § R33` Result-Shape Wrapping (envelope spec + reference handler)
- Action: enforced_handler must wrap non-dict returns into `{"status":"success","data": ...}` envelope per R33 spec
- VERIFY: `grep -A5 'def _wrap' src/<pkg>/core/tool_annotations.py` shows the wrapper logic
- GATE: R33 wrap function present in install_annotator
- STOP-IF-FAIL: without R33, FastMCP throws ValueError on non-dict returns and breaks tool packaging

**STEP 4.3** — Contract Enforcer wired (G6 banned-tool block)
- CONSULT: Bible `07_PAIRING_CONTRACT.md § 5 G6` (banned-tool dispatch behavior + -32006 spec)
- Action: server.py `from .core.contract_enforcer import get_enforcer; _enforcer = get_enforcer()` at module init
- VERIFY: probe a banned tool with `{}` args; expect JSON-RPC `-32006` + migration message
- GATE: 5/5 banned tools return -32006

**STEP 4.4** — Capability adds MUST pass R29 verification per tool
- CONSULT: Bible `14_*.md § R29 Law 11` + § R31 Per-tool VERIFY GATE
- For each new tool in capability_adds_*.py:
  - STEP 4.4.a — write the tool implementation
  - STEP 4.4.b — invoke it against live target with realistic args (or cite vendor docs URL in source comment)
  - STEP 4.4.c — record `verified_at: <date>` in contract.yaml `tool_inventory` per-tool list
- STOP-IF-FAIL: if a tool's underlying SQL function / REST endpoint can't be verified, DO NOT register it. Pattern-inferred guesses (e.g., `SHOW <FEATURE> <NOUN>` extrapolated from another feature) are BANNED per R29.
- GATE: every tool in capability_adds_*.py has either verified_at timestamp OR vendor docs URL in source comment

**STEP 4.5** — G8 redact_processor in structlog chain
- CONSULT: Bible `07_PAIRING_CONTRACT.md § 5 G8` (redact patterns + processor chain spec)
- Action: insert `g8_redact_processor` in structlog.configure processor chain BEFORE `JSONRenderer`
- VERIFY: log a test event with a credential-named field (e.g., `<vendor>_private_key`, `api_key`, `password`); assert log line shows `***REDACTED***`
- GATE: redact_processor verified active

**STEP 4.6** — register_elicitation + register_sampling called (Check 12.4)
- CONSULT: Bible `03_MCP_SERVER_STANDARD.md § 6.1` MCP Primitives Registration
- Action: server.py register_primitives() invokes both
- VERIFY: container startup log shows `elicitation_registered` + `sampling_registered`
- GATE: both calls execute without error

**STEP 4.7** — Static-arity audit (R45 OR R49 per pair architecture; v19.1.9 / v19.1.11)
- CONSULT: Bible `16_FRAMEWORK_REFINEMENTS_v19.1.9.md` (R45 two-layer) OR `18_FRAMEWORK_REFINEMENTS_v19.1.11.md` (R49 all-in-one-class)
- Action: per pair architecture (declared in contract.yaml § Architecture from STEP 2.4), run the corresponding audit script:
  - Two-layer: `python3 framework/scripts/wrapper_arity_audit.py --server-file src/<pair>_mcp/server.py --client-file src/<pair>_mcp/core/api_client.py --client-symbol <pair>_client --client-class <Pair>APIClient`
  - All-in-one-class: `python3 framework/scripts/intra_class_arity_audit.py --class-file src/<pair>_client_complete.py --class-name <Pair>Client`
- VERIFY: audit script exits 0 (or `--soft-warn` first cycle)
- GATE: 0 issues OR soft-warn cycle with backlog tracked
- STOP-IF-FAIL: per R47 impl-widening recipe, fix at the impl side (add named-optional alias or `**kwargs`), do NOT rewrite all callers

---

### PHASE 4.8 — Guardrail Tests

**STEP 4.8.1** — Author tests/guardrails/test_guardrails.py
- CONSULT: Bible `07_PAIRING_CONTRACT.md § 5` + `09_VERIFICATION_PROTOCOL.md Check 4`
- Action: write tests covering G1-G11 + Check 12 subchecks + pair metadata
- Action: G2 test asserts `enabled: false` + REMOVED rationale (G2 removed v19.1.2)
- Action: G5/G9 tests assert disabled-with-rationale OR enabled-with-config per pair
- VERIFY: `pytest tests/guardrails/ -q` (run inside container if possible)
- GATE: all guardrail assertions pass

---

### PHASE 5 — Skill Upgrade

**STEP 5.1** — Update skill frontmatter (Bible §04 §3 — mandatory pairing fields)
- CONSULT: Bible `04_SKILL_STANDARD.md § 2` (frontmatter schema)
- Action: name, description, version, mcp_server, mcp_tool_prefix, mcp_tool_count, mcp_auth, mcp_endpoint, pairing_contract_version, pairing_contract_path
- VERIFY: `head -15 /root/.claude/skills/<slug>/SKILL.md | grep -cE '^(version|mcp_server|mcp_tool_prefix|mcp_tool_count|mcp_auth|mcp_endpoint|pairing_contract_version|pairing_contract_path):'` returns 8
- GATE: 8/8 mandatory fields present

**STEP 5.2** — Skill body has 9 mandatory sections
- CONSULT: Bible `04_SKILL_STANDARD.md § 3` (9 mandatory sections)
- Action: ensure Identity, Pairing Block, When to Invoke, Tool Map, Pre-Call Sequences, Guardrails (mirror), Verification Rituals, Cascade Notes, Gotcha Registry (≥5)
- VERIFY: `grep -cE '^## (Identity|Pairing Block|When to Invoke|Tool Map|Pre-Call Sequences|Guardrails|Verification Rituals|Cascade Notes|Gotcha Registry)' /root/.claude/skills/<slug>/SKILL.md` returns 9
- GATE: 9/9 sections present

**STEP 5.3** — Mirror skill byte-identical (R11 dual-mirror)
- Action: `diff /root/.claude/skills/<slug>/SKILL.md /home/claude/.claude/skills/<slug>/SKILL.md` (note: in this VPS, `/home/claude` is symlinked/hardlinked, so should auto-match)
- GATE: zero diff output

**STEP 5.4** — R11 cleanup: no `skill/` subdir inside MCP repo
- VERIFY: `ls /opt/mcp-servers/<slug>/skill/ 2>/dev/null` returns nothing OR documented archive
- GATE: skill content lives ONLY in `/root/.claude/skills/<slug>/`

---

### PHASE 7 — Build + Tier-appropriate Supply Chain

**STEP 7.1** — Build image with new tag
- Action: `docker compose build <slug>` then `docker tag <slug>-<slug>:latest <slug>-<slug>:v<NEW>`
- VERIFY: `docker images | grep <slug>-<slug>:v<NEW>` shows the new tag
- GATE: image built + tagged

**STEP 7.2** — Tier-appropriate scan (R16)
- CONSULT: Bible `06_STACK_MANIFEST.md § 1.6.1` Supply-Chain Tier Gating
- Tier 1: `trivy image --severity HIGH,CRITICAL --exit-code 0 <slug>:v<NEW>` (informational; HIGH OS-deps without upstream fix acceptable)
- Tier 2+: trivy + SBOM (syft) + digest-pin
- Tier 3: + cosign + SLSA-L3
- VERIFY: scan output captured to `docs/trivy-scan-<DATE>.txt`
- GATE: tier-appropriate artifacts present
- STOP-IF-FAIL (Tier 2+): if any HIGH with available upstream fix, MUST patch before SIGN

---

### PHASE 9 — Production Cutover

**STEP 9.1** — Pre-cutover .env hash check (final)
- Action: `NEW=$(sha256sum .env | awk '{print $1}'); BASE=$(cat /tmp/<slug>-baseline-*/env-hash.txt | awk '{print $1}'); [[ "$NEW" == "$BASE" ]] || exit 1`
- GATE: .env unchanged from Phase 0.2 baseline

**STEP 9.2** — Deploy
- Action: `IMAGE_TAG=v<NEW> docker compose up -d`
- VERIFY: `docker inspect <slug> --format='{{.State.Health.Status}}'` returns healthy within 60s
- GATE: container healthy

**STEP 9.3** — Phase 9 Completion Gate (1–17 checks per current Bible)
- CONSULT: Bible `09_VERIFICATION_PROTOCOL.md § 1` (full 1–18 check list with pass criteria; Checks 13–18 specified in linked refinement chapters) + Bible `18_FRAMEWORK_REFINEMENTS_v19.1.11.md § Check Renumber` (Check 13 collision resolution: R45 → 16, R49 → 17) + Bible `19_FRAMEWORK_REFINEMENTS_v19.1.12.md § Check 18` (R50 per-tool timeout enforcement)
- Run every check 1–17 (or N/A with Tier 1 rationale)
- VERIFY: each check's command returns the expected GREEN result
- Per-pair MUST checks (apply to ALL pairs; concrete probe targets pair-specific):
  - `/health` returns `{status, version, framework, contract.pair_status: SIGNED}`
  - `tools/list` count matches `contract.target_count`
  - All tools have `outputSchema` + `annotations` (Check 12.1+12.2)
  - `MCP-Protocol-Version: 2025-11-25` negotiated on initialize
  - 5 banned tools return -32006 with migration message
  - Identity probe (per Pairing Contract §10) returns expected account/org marker
  - Static-arity audit clean (Check 16 if two-layer / Check 17 if all-in-one-class)
  - Symmetric reader/writer audit clean (Check 15) where R48 applies
- GATE: 17/17 GREEN (N/A allowed only where R16 explicitly permits)

**STEP 9.4** — R32 4-layer Census re-run on FULL post-deploy tool surface
- CONSULT: Bible `14_*.md § R32`
- Action: `python3 scripts/tool_health_census.py --rate-limit-sleep 0.05` (against the new deployment)
- Triage every BROKEN row per Phase 0.5.3 rubric
- GATE: 0 TRUE_BUG entries (PROBE_GAP and FEATURE_NOT_ENABLED acceptable)
- STOP-IF-FAIL: if TRUE_BUG > 0, ROLLBACK + fix + redeploy

**STEP 9.5** — Flip contract pair_status to SIGNED
- Action: `sed -i 's/pair_status: DRAFT/pair_status: SIGNED/' contract.yaml` then rebuild + redeploy with same image tag
- VERIFY: `/health.contract.pair_status` returns SIGNED
- GATE: contract SIGNED + surfaced

---

### PHASE 10 — Closure (within 48h post-cutover)

**STEP 10.1** — Pristine Sweep (7 categories)
- CONSULT: Bible `09_VERIFICATION_PROTOCOL.md § 3` (full 7-category sweep spec)
- Cleanup orphan /tmp/ session artifacts (keep only baseline + cutover dirs through 30d retention)
- Verify skill mirrors byte-identical
- Verify .env final hash unchanged
- Verify R13 cascade alignment across both scales
- GATE: 7-category sweep clean

**STEP 10.2** — Pilot learnings written
- CONSULT: Bible `09_VERIFICATION_PROTOCOL.md Check 11`
- Action: write `<pair-repo>/docs/pilot-learnings.md` with: final state, what went well, what surprised, framework refinement candidates (R<N> for next Bible bump)
- GATE: pilot-learnings.md committed

**STEP 10.3** — Vault writeback + session-learnings + ST decision
- Action: `mcp__vault-mcp__vault_submit_knowledge` with pair_slug + framework_version + outcome
- Action: append `[UPGRADE] <slug> Bible-v<X> server-v<Y>` to session-learnings.md with non-obvious learnings
- Action: `mcp__sequential-thinking__record_decision` with tags=mcp-upgrade,<pair>,signed,st-derived + calibrated confidence
- GATE: all three persistence sinks updated

**STEP 10.4** — Universal Completion Gate (per CLAUDE.md)
- Functional verification / Tests passing / Documentation updated / No regressions / .env preserved / Cleanup complete / Decision recorded with calibrated confidence
- GATE: 7/7 UCG GREEN

**STEP 10.5** — Bible upstream cascade (if new pattern discovered)
- If the upgrade surfaced a new generalizable pattern (R<N>): author a PR or DRAFT entry to the Bible repo (`framework/scripts/...` for tooling, `XX_FRAMEWORK_REFINEMENTS_vX.Y.Z.md` for refinements)
- VERIFY: PR opened or DRAFT pushed to Bible main
- GATE: new pattern is captured at the framework level so the next pair benefits

---

### Hard rules (apply at every step, never overridable)

1. **Never proceed past a failed gate** — fix the failure or rollback. No silent workarounds.
2. **Never guess at upstream API specs** (R29) — verify or cite docs URL.
3. **Never skip Phase 0.5 Census** — it's how you find pre-existing baseline bugs that wrapping doesn't fix.
4. **Never ship a tool without R31 verification** — `verified_at` timestamp OR vendor docs URL in source.
5. **Never modify .env** — Pre-flight Rule 0 absolute.
6. **Never mark SIGNED without 17/17 + R32 census = 0 TRUE_BUG**.
7. **Never duplicate Bible content into this skill** — replace with smart reference.

## Validation (quick-check)

```bash
# Ten Laws quick check (per Bible §01)
grep -rn "__new__" src/ >/dev/null && \                               # Law 1
grep -rn "run_in_executor" src/ | wc -l | grep -q "^0$" && \          # Law 3
grep -rnE "\\*\\*kwargs" src/tools/ | wc -l | grep -q "^0$" && \      # Law 5
grep -rn "stateless_http=True" src/ >/dev/null && \                   # Law 6
grep -rn "OAuthProvider" src/ | wc -l | grep -q "^0$" && \            # Law 7 pattern-2
grep -rn 'on_duplicate="error"' src/ >/dev/null && \                  # fail-fast
echo "Ten Laws: PASS"

# Pairing Contract present (Bible §07 — Law 8)
test -f <pair-repo>/docs/pairing-contract.md && echo "Contract: in pair repo"
ls tests/guardrails/test_g*.py | wc -l | grep -qE "^(10|11)$" && echo "Guardrails: tests present"   # Law 9

# Check 12 — MCP 2025-11-25 spec-compliance (Bible §09 Check 12)
curl -fsS -X POST https://<pair>-mcp.arjtech.in/mcp -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}' | \
  jq '.result.tools | map(.outputSchema != null) | all'                # 12.1 — all have outputSchema
grep -E 'elicitation.*\.register\(mcp\)' src/server.py                  # 12.4

# Two-scale cascade (R13)
EXPECTED_SERVER=v1.0.0; EXPECTED_FRAMEWORK=v19.1.12; PAIR=<slug>
bash /root/aj-workspace/scripts/cascade-check.sh $EXPECTED_SERVER $EXPECTED_FRAMEWORK $PAIR

# Static-arity audit (R45 if two-layer; R49 if all-in-one-class)
python3 framework/scripts/wrapper_arity_audit.py --server-file ... --client-file ...   # OR
python3 framework/scripts/intra_class_arity_audit.py --class-file ... --class-name ...

# Post-deploy drift (event-driven, NOT calendar cron — R14)
DRY_RUN=1 /root/aj-workspace/scripts/drift-check.sh --pair=<slug>
```

## Common Mistakes — quick triage (full list lives in Bible chapters)

For the full anti-pattern catalog, consult: Bible `00_PHILOSOPHY.md` (principles), `01_INVIOLABLE_LAWS.md` (laws + Common Mistakes table), `14_*.md § R29` (upstream-verification banned patterns). High-frequency triage:

| Symptom | Where to look in Bible |
|---|---|
| `TypeError: takes N positional arguments but M were given` | `16_*.md` (R45 two-layer) or `18_*.md` (R49 all-in-one-class) |
| `ValueError: structured_content must be a dict or None` | `14_*.md § R33` Result-Shape Wrapping |
| `ImportError: FastMCP background tasks require the 'tasks' extra` | `06_STACK_MANIFEST.md` (use `fastmcp[tasks]>=3.0.2`) |
| Cascade-check fails on framework_version mismatch | `14_*.md § R13` (atomic surface bump) |
| Blanket 401 after auth change | `01_INVIOLABLE_LAWS.md § Law 7` (Pattern 2 + OAuthProvider ban) |
| Tool returns success on REST 4xx/5xx | semantic gate caught Codex-class find — wrapper boundary status propagation. No AST audit detects this; rely on R39 + Codex review. |
| Operator-intent tool needs N round-trips | `17_FRAMEWORK_REFINEMENTS_v19.1.10.md § R48` Complete-Capability Invariant |
| Symmetric metadata reader missing for writer | `17_*.md § Check 15` Symmetric Reader Audit |
| Container `.bible` label lags Bible main | this skill's Bible-pull discipline + Phase 1 Bible-sync minor patch |

## Workflow Integration

- **Pristine Sweep**: 7 categories per Bible `09_VERIFICATION_PROTOCOL.md § 3`, post-upgrade.
- **Completion Gate**: 17 checks per Bible `09 § 1` (post v19.1.11 renumber); ALL must pass for Contract to SIGN.
- **Decision journal**: every upgrade records `mcp__sequential-thinking__record_decision` with `tags: mcp-upgrade,<pair>,<outcome>,st-derived` per Tier-1 Directive 5.
- **Vault writeback**: completion → `vault_submit_knowledge` with pair slug + framework version.
- **Session-learnings**: post-upgrade append `[UPGRADE] <pair> Bible-v<X> server-v<Y>` entry with non-obvious learnings.

## Session Operating Rules — zero lapses

1. **Pull canonical Bible first** (this skill's prologue protocol). Container labels and skill copies are not sources of truth.
2. **Skill = orchestration, Bible = content.** Every CONSULT line is a smart reference to Bible. Never restate Bible content in this skill.
3. **Pair-repo discipline** — all pair-specific artifacts in pair repo (`docs/pairing-contract.md`, `docs/pilot-learnings.md`, `docs/upstream-sync-<date>.md`). Never pollute the Bible.
4. **Two-scale version clarity** — always state which scale you're bumping (server vs framework). Never conflate.
5. **Zero-deferred-must-adopt** — sync recommendations get implemented this upgrade or waived with expiry.
6. **Cascade-check post-bump** — run `cascade-check.sh` after any version edit. All surfaces must align across both scales.
7. **Gate before SIGN** — Phase 9 Completion Gate 17/17 GREEN is the only path to `pair_status: SIGNED`.
8. **Tier-appropriate, not tier-maximum** — R16 banned gold-plating (Tier 3 requirements on Tier 1 pair).
9. **Event-driven drift only** — no nightly cron for stable production pairs (R14).
10. **Post-cutover Phase 10** — quality audit within 48h. Findings → `<pair-repo>/docs/pilot-learnings.md` + framework refinements upstream-cascaded to Bible if they generalize.
11. **Never claim done without running the Gate** — cheerleading is a drift class. Every `PASS` needs a command output.
12. **Skill ↔ Bible sync** — when Bible bumps, this skill's version markers + Refinements Map + Bible chapter map update within the same session. Drift = paired-unit failure.
