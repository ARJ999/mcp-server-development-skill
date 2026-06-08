---
name: mcp-server-development
description: "Use when developing a new MCP server from scratch, upgrading an existing MCP server, performing a quality audit, or pairing a skill with its MCP. This skill is the ORCHESTRATION LAYER — operational playbook, decision flow, smart references — that consults the canonical Bible repo (https://github.com/ARJ999/MCP-God-Agent-Development-Bible) for every rule, refinement, and detail. Zero content duplication: rules live in the Bible; this skill tells you when, in what order, and where to look. Bible v19.1.12 canonical (R25-R50, 18-check Completion Gate). Reference pilots: Fivetran (SIGNED, production), tableau-mcp v6.0.1 (R45 pilot), salesforce-mcp v7.1.1 (R48 + R50 pilots), snowflake-mcp v22.0.5 (R49 pilot). Pair-specific artifacts live in each pair's own repo under docs/, not in the Bible."
version: 4.2.2
---

<!-- Changelog (skill-resident, orchestration evolution only):
v4.2.2 (2026-06-08) — Major internal-consistency reconciliation to Bible v19.1.12 (CCVS gate hardening).
                     R-number citations fixed to live Bible 14 (census R39, result-shape R40, upstream
                     R38 parent R30 — NOT R29/R31/SERF). Authored + committed `cascade-check.sh`
                     (`--static`, fail-closed, >=1 surface/scale) in aj-workspace. SIGN lifecycle made
                     Bible-correct: verify -> sign -> final 18-check gate ALL run on STAGING (Phase 8)
                     against the v<NEW> image .Id; Phase 9 PROMOTES the already-verified+signed candidate
                     (prod: old-SIGNED -> new-SIGNED directly, never DRAFT). Isolated upgrade WORKTREE so
                     the live mounted contract is never mutated mid-upgrade; both mirrors (.yaml + .md)
                     signed in lockstep via canonical whole-line set; full rollback (reset --hard
                     pre-upgrade-commit + redeploy pre-upgrade image). Naming legend + real estate shapes;
                     two-scale placeholders de-conflated; restored Phase 0-NEW + Phase 6 (event-driven
                     drift R14). Conditional primitives (§03 §6); Bible-canonical banned-tool fields
                     (replacement/ban_scope); structured (non-grep) verification; Check 18 = manual
                     four-condition audit (wait_for_audit.py deferred to v19.1.13).
v4.2.0 (2026-05-16) — Bible v19.1.12 canonization sync. AJ zero-ambiguity directive collapsed
                     v19.1.11 DRAFT + v19.1.12 DRAFT into single canonical v19.1.12 on Bible main
                     (R49 + R50 + Check 16/17/18). Updated Framework version field, Refinements Map
                     (added row 19 + marked rows 18/19 CANONIZED), Smart Reference Index, pair table
                     (salesforce v7.1.0 → v7.1.1 + R50 pilot attribution; snowflake R49 DRAFT → CANONIZED),
                     Phase 9 references (17-check → 18-check), STEP 4.7 R45/R49 selector wording (DRAFT → CANONIZED).
v4.1.0 (2026-05-06) — 57% refactor: 1616→694 lines. Removed ~1100 lines of Bible-duplicated content
                     (R38 upstream-verification spec, Ten Laws table, Law 7/11, R32/R40, Pairing Contract detail,
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

**Drift detection**: at session start, agent compares this skill's "Framework version" vs the canonical Bible version (`grep -m1 'Latest refinement' /root/aj-workspace/MCP-God-Agent-Development-Bible/README.md`). Mismatch → run sync pass on this skill before doing pair work.

**Anti-pattern**: copying Bible content into this skill. If a section feels duplicative of Bible material, replace it with a pointer (`see Bible §N` or `Bible chapter file_X.md § Y`). Skill content = orchestration + decision logic + smart references. Bible content = rules + refinements + specs.

## What Goes Where — Litmus Test (apply on EVERY change, ongoing)

Every time anything is added, modified, or refined, run it through this test BEFORE writing it anywhere:

### Bible-resident content (canonical, cited, framework-wide)

A piece of content goes in the Bible if it is:

- **A rule, law, refinement, or check** with a stable identifier (Lxx, Rxx, Cxx, §N) that gets CITED elsewhere
- **A specification** that defines WHAT MUST be true (schema, contract shape, MCP wire-format requirement)
- **A template** that pairs copy from (`templates/skill-template/SKILL.md`, `templates/pairing-contract-template.md`)
- **A reference script** that pairs invoke as-is (`framework/scripts/wrapper_arity_audit.py`, `symmetric_reader_audit.py`; `intra_class_arity_audit.py` is **authoring-pending** — until it lands, run the R49 intra-class audit MANUALLY per Bible §18 pseudocode, per the Quick-Map note)
- **A cross-pair invariant** — applies to every MCP server identically regardless of platform
- **An anti-pattern catalog** — what is BANNED and why (R38 hallucinated-SQL ban, Pattern 2 + OAuthProvider, etc.)
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
| "Run R49 audit at Phase 8.4 Check 17 if all-in-one-class" | Skill (Operational Playbook STEP 4.7 + STEP 8.4) | Orchestration — when/how to apply R49 |
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
| [`14_FRAMEWORK_REFINEMENTS_v19.1.7.md`](https://github.com/ARJ999/MCP-God-Agent-Development-Bible/blob/main/14_FRAMEWORK_REFINEMENTS_v19.1.7.md) | R25–R40 (R38–R40 additive in this same v19.1.7 file) | CANONIZED 2026-04-22 | Cascade R13, drift R14, Census R39, Result-shape R40, Upstream-Verification R38 (parent R30) |
| [`15_FRAMEWORK_REFINEMENTS_v19.1.8.md`](https://github.com/ARJ999/MCP-God-Agent-Development-Bible/blob/main/15_FRAMEWORK_REFINEMENTS_v19.1.8.md) | R41–R44 (salesforce-mcp v7.0.0 pilot) | CANONIZED | Cross-cutting feature gating (FEATURE_NOT_ENABLED unification + Pattern A platform-managed transport) |
| [`16_FRAMEWORK_REFINEMENTS_v19.1.9.md`](https://github.com/ARJ999/MCP-God-Agent-Development-Bible/blob/main/16_FRAMEWORK_REFINEMENTS_v19.1.9.md) | R45–R47 (tableau-mcp v6.0.1 pilot) | CANONIZED 2026-05-03 | **Wrapper-impl arity discipline** — R45 AST audit (two-layer pattern) + R46 kwargs-by-name + R47 impl-widening recipe |
| [`17_FRAMEWORK_REFINEMENTS_v19.1.10.md`](https://github.com/ARJ999/MCP-God-Agent-Development-Bible/blob/main/17_FRAMEWORK_REFINEMENTS_v19.1.10.md) | R48 + Check 15 (salesforce-mcp v7.1.0 pilot) | CANONIZED 2026-05-06 | **Complete-Capability Invariant** — operator-intent tools deliver in one round-trip + symmetric metadata reader/writer audit |
| [`18_FRAMEWORK_REFINEMENTS_v19.1.11.md`](https://github.com/ARJ999/MCP-God-Agent-Development-Bible/blob/main/18_FRAMEWORK_REFINEMENTS_v19.1.11.md) | R49 + Check renumber (snowflake-mcp v22.0.5 pilot) | CANONIZED 2026-05-16 | **Intra-class arity audit** — sibling to R45 for all-in-one-class servers + Check 13 collision resolution (R45 → 16, R49 → 17) |
| [`19_FRAMEWORK_REFINEMENTS_v19.1.12.md`](https://github.com/ARJ999/MCP-God-Agent-Development-Bible/blob/main/19_FRAMEWORK_REFINEMENTS_v19.1.12.md) | R50 + Check 18 (salesforce-mcp v7.1.1 pilot) | CANONIZED 2026-05-16 | **Per-tool timeout enforcement** — `asyncio.wait_for` wrap at `install_annotator` boundary with category-aware caps; resolves 665+ unbounded `asyncio.to_thread` sites |

**Architecture-aware static-arity audit selection** (R45 + R49 are mutually exclusive per pair):

| Server pattern | Audit | Detect by |
|---|---|---|
| Two-layer FastMCP (`@mcp.tool()` wrapper + separate `api_client` class) | **R45** — `wrapper_arity_audit.py` | Separate `server.py` + `core/api_client.py`; wrapper calls `<client>.<method>(...)` |
| All-in-one-class (single class file, intra-class dispatch) | **R49** — `intra_class_arity_audit.py` (**authoring-pending**: until the script lands, run the intra-class arity audit MANUALLY per Bible §18 pseudocode) | One large class file; dispatch is `self.<method>(...)` |
| Single-method / fully-self-contained wrappers | Neither (rely on R39 + Codex) | No `self.<method>` or `<client>.<method>` indirection inside `@mcp.tool()` body |

R39 (realistic-args census) and Codex hard-gate (cross-model review) apply unconditionally regardless of pattern. R50 (per-tool timeout enforcement at `install_annotator`) applies unconditionally to every FastMCP-based pair regardless of architecture.

**Migration path for pairs still on v19.1.6 or earlier**: adopt R25–R50 in Bible-version order. Full R25–R50 adoption expected for any major-version upgrade post 2026-05-16. Pair-side `phase9_completion_gate.sh` templates need mechanical Check renumber (R45 Check 13 → 16, add Check 17 if all-in-one-class, add Check 18 for all FastMCP pairs) on next refresh.

## What is a "Pair"? — Definitive Glossary (read this BEFORE the playbook)

A **pair** = one MCP server + one paired skill, governed by ONE Pairing Contract. They ship together, version together, get upgraded together. Neither half exists in production without the other.

```
                    ┌───────────────── PAIR ─────────────────┐
                    │                                         │
   /opt/mcp-servers/<slug>-mcp/   ←─ Pairing Contract ─→  /root/.claude/skills/<slug>/
   (the MCP server,                                      (the skill,
    Docker container,                                     SKILL.md (self-contained),
    contract.yaml,                                        domain expertise,
    src/, scripts/, tests/)                               gotcha registry)
                    │                                         │
                    └─ together = ONE pair = SIGNED unit ─────┘
```

**Reference pairs on this VPS** — ⚠️ **DIRECTIONAL EXAMPLES, NOT live state.** Server versions, tool counts, and framework levels DRIFT, so this skill does NOT store them — ALWAYS derive current values from the live surface (`docker ps` / `/health` / `tools/list` / each pair's `contract.yaml framework_version`). The table lists ONLY the stable identity + static-arity pattern:

| Pair slug | MCP host | Paired skill | Static-arity pattern |
|---|---|---|---|
| `snowflake` | snowflake-mcp.arjtech.in | `/root/.claude/skills/snowflake/` | R49 (all-in-one-class) — pilot |
| `fivetran` | fivetran-mcp.arjtech.in | `/root/.claude/skills/fivetran/` | TBD per pattern |
| `salesforce` | salesforce-mcp.arjtech.in | `/root/.claude/skills/salesforce/` | R45 (two-layer); R48 + R50 pilots |
| `tableau` | tableau-mcp.arjtech.in | `/root/.claude/skills/tableau/` | R45 (two-layer) — pilot |
| `blue` | blue-mcp.arjtech.in | `/root/.claude/skills/blue/` | TBD per pattern |
| `aws` | aws-mcp.arjtech.in | `/root/.claude/skills/aws/` | Registry-dispatch (R50 runtime applies via install_annotator wrap; registry-dispatch formalization deferred to v19.1.13 per PR #25) |

> To check whether any pair lags canonical Bible v19.1.12, DERIVE it LIVE — compare each pair's `contract.yaml framework_version` (or `/health.framework`) against the canonical Bible version. This page deliberately records NO per-pair drift state (it would stale and contradict the source-of-truth discipline).

### Two distinct invocation modes — DO NOT CONFUSE

| When the user asks for... | Invoke this skill | Why |
|---|---|---|
| Domain work using a specific platform (e.g., "query Snowflake", "list Fivetran connectors") | The **pair skill** (e.g., `snowflake`, `fivetran`) | Pair skill carries domain expertise + tool inventory + gotchas |
| Build, upgrade, audit, migrate, or modify a pair's MCP+skill itself | **`mcp-server-development`** (this skill) | This skill carries the orchestration playbook; it consults the Bible for rules |

**Concrete example**: AJ asks "upgrade snowflake to Bible v19.2" → invoke `mcp-server-development` (the upgrade playbook), which consults Bible chapters in sequence, AND coordinates the upgrade of BOTH the snowflake MCP server (`/opt/mcp-servers/snowflake-mcp/`) AND the snowflake skill (`/root/.claude/skills/snowflake/`) IN LOCKSTEP. The pair stays as a unit.

### Pair invariants (never violated, ever)

1. **Lockstep version + tool-count sync** — (a) version lockstep (per cycle): the SERVER artifact tag — `contract.yaml mcp_image_tag` (the canonical Bible §07 field; the `:tag` portion) == compose image tag `v<NEW>` == `/health.version` — move together each upgrade. The skill's own semver `version` advances with the pair (its MAJOR tracks the paired MCP per Bible §04) but need NOT string-equal that tag. NOTE: Pairing Contract `contract_version` is the contract-SCHEMA/binding version (e.g. `1.0`) — it stays FIXED across normal pair upgrades (skill `pairing_contract_version` tracks it) and is NOT part of this lockstep. (b) tool-count sync (a SEPARATE axis — a count, not a version): `tool_inventory.target_count` == live `tools/list` count == skill `mcp_tool_count`. Drift on either = Phase 5 + R13 cascade-check failure.
2. **Single source of truth** — Pairing Contract lives in pair-repo `<pair-repo>/docs/pairing-contract.md`. Skill `pairing_contract_path` frontmatter points to it. Bible NEVER carries pair-specific contract content.
3. **Skill mirror** — `/root/.claude/skills/<slug>/SKILL.md` and `/home/claude/.claude/skills/<slug>/SKILL.md` are byte-identical (R11 dual-mirror). On this VPS they're symlinked/hardlinked, so this is automatic.
4. **R11 cleanup** — pair MCP repo MUST NOT contain `skill/` subdirectory. Skill content lives ONLY in the global skills dir.
5. **Pair upgrade is atomic** — never bump server version without bumping skill `mcp_tool_count` in same cycle. Cascade-check (R13) verifies surfaces align across both version scales.

### When a NEW pair is being built (not upgrade — net-new MCP)

Same skill (`mcp-server-development`), but the runbook **enters at Phase 0-NEW** (Net-New Pair Bootstrap), NOT the upgrade Phase 0 pre-flight. The upgrade Phase 0 assumes an already-live pair (existing container, `.env`, rollback image, backup branch) — none of which exist for a from-scratch build. Phase 0-NEW scaffolds those from the Bible templates (`templates/skill-template/SKILL.md`, `templates/pairing-contract-template.md`) and builds the first server.py per Bible `03_MCP_SERVER_STANDARD.md`, then **converges into the shared pipeline at Phase 1.5** (Upstream Capability Sync) → Phase 2 → … → Phase 9. Bible `08_UPGRADE_PLAYBOOK.md` is upgrade-oriented; the from-scratch scaffolding sequence is skill-resident orchestration (Phase 0-NEW below).

## Bible Chapter Quick-Map (smart references — single source of routing)

When the Operational Playbook says `CONSULT § <number>` or `CONSULT § <name>`, the chapter is in `ARJ999/MCP-God-Agent-Development-Bible` repo, file `0N_<TOPIC>.md`. This is the only routing table — never duplicated, never restated.

| Bible chapter | Skill playbook step that uses it |
|---|---|
| `00_PHILOSOPHY.md` | Onboarding — read once for principles |
| `01_INVIOLABLE_LAWS.md` | Phase 4 (every law check); contains Ten Inviolable Laws + Law 7 5-pattern auth + R38 Upstream Verification Mandate (parent R30) |
| `02_ARCHITECTURE.md` | Phase 2/3 (4-Tier v3.1 + R7 Triad Trigger + R13 cascade discipline) |
| `03_MCP_SERVER_STANDARD.md` | Phase 4.1–4.3 (server.py + install_annotator + Contract Enforcer + MCP Primitives Registration § 6 + Check 12 Implementation Patterns) |
| `04_SKILL_STANDARD.md` | Phase 5 (skill upgrade — frontmatter, 9 sections, dual-mirror) |
| `05_TOOL_STANDARD.md` | Phase 4.4 (tool design + outputSchema + ToolAnnotations + Task R24 + R38 upstream-verification + R40 result-shape) |
| `06_STACK_MANIFEST.md` | Phase 3 (Gold Stack pin bumps + Supply-Chain Tier Gating R16) |
| `07_PAIRING_CONTRACT.md` | Phase 2 (12-section contract.yaml + Core 8 + Domain-Conditional 2 guardrails) |
| `08_UPGRADE_PLAYBOOK.md` | Phase 1, 6, 8, 9 — full 9-Phase Bible-side upgrade detail (companion to this skill's Operational Playbook; Phase 6 invocation-wiring + Phase 8 staging-canary specs) |
| `09_VERIFICATION_PROTOCOL.md` | Phase 0.5 (Census) + 8.4 (18-Check Completion Gate on SIGNED staging, BEFORE promotion) + 10 (7-category Pristine Sweep) |
| `10_DOMAIN_COVERAGE_STANDARD.md` | Phase 1.5 (capability gap analysis) |
| `11_EVALUATION_HARNESS.md` | Phase 8 (paired-vs-unpaired evaluation, Tier 2+) |
| `12_AUTORESEARCH_BINDING.md` | Phase 5 (skill optimization loops) |
| `13_UPSTREAM_SYNC_PROTOCOL.md` | Phase 1.5 mandatory 6-step sync |
| `14_FRAMEWORK_REFINEMENTS_v19.1.7.md` | Throughout — R25–R40 (cascade R13, drift R14, Census R39, Result-shape R40, Upstream-Verification R38 parent R30; R38–R40 additive in this file) |
| `15_FRAMEWORK_REFINEMENTS_v19.1.8.md` | Throughout — R41–R44 (FEATURE_NOT_ENABLED unification, Pattern A transport) |
| `16_FRAMEWORK_REFINEMENTS_v19.1.9.md` | Phase 4 + Phase 8.4 Check 16 — R45–R47 (wrapper-impl arity, two-layer) |
| `17_FRAMEWORK_REFINEMENTS_v19.1.10.md` | Phase 4 + Phase 8.4 Check 15 — R48 (Complete-Capability Invariant) |
| `18_FRAMEWORK_REFINEMENTS_v19.1.11.md` | Phase 4 + Phase 8.4 Check 17 — R49 (intra-class arity, all-in-one-class) |
| `19_FRAMEWORK_REFINEMENTS_v19.1.12.md` | Phase 4 + Phase 8.4 Check 18 — R50 (per-tool timeout enforcement at install_annotator) |
| `templates/pairing-contract-template.md` | Phase 2 (blank Contract template — copy + fill) |
| `templates/skill-template/SKILL.md` | Phase 5 (blank skill scaffold for new pairs) |
| `framework/scripts/wrapper_arity_audit.py` | Phase 8.4 Check 16 (R45 enforcement, two-layer servers) |
| `framework/scripts/intra_class_arity_audit.py` (**authoring pending** — Bible R49 ships the AST pseudocode; canonical script "to be authored") | Phase 8.4 Check 17 (R49 enforcement, all-in-one-class). Until the script lands, run the intra-class arity audit MANUALLY per the Bible §18 pseudocode (snowflake-mcp v22.0.5 pilot did this) |
| `19_*.md § Check 18` (R50 spec; **no script dependency in v19.1.12**) | Phase 8.4 Check 18 — R50 **manual four-condition audit**: (1) install_annotator wraps via `asyncio.wait_for`, (2) every annotated `to_thread` flows through that wrap, (3) contract.yaml `guardrail_timeout.enforcement_layers` declares tool-layer + session-layer, (4) source pointers match. `framework/scripts/wait_for_audit.py` is the *future automation* (authoring deferred to v19.1.13 candidate); until then the gate runs by manual/grep verification of conditions 1–4. Runtime guard live in salesforce-mcp v7.1.1. |
| `framework/scripts/symmetric_reader_audit.py` | Phase 8.4 Check 15 (R48 enforcement) |

### How to fetch Bible content

The **mandatory, canonical path** is the local-clone pull (option 1 below) — it is the ABSOLUTE PROTOCOL at the top of this skill. The `gh` options below are read-only conveniences that **supplement, never replace** that pull (use them only AFTER the pull, e.g. to confirm sha parity or peek at a single chapter):

1. `cd /root/aj-workspace/MCP-God-Agent-Development-Bible && git pull --ff-only origin main` then read locally (**MANDATORY** — the source-of-truth path; required before referencing any Lxx/Rxx/Cxx/§; preferred for multi-chapter consultation)
2. `gh repo view ARJ999/MCP-God-Agent-Development-Bible --json defaultBranchRef` — confirm the latest sha matches your freshly-pulled clone (supplement)
3. `gh api repos/ARJ999/MCP-God-Agent-Development-Bible/contents/<FILE>.md --jq .content | base64 -d` — quick single-chapter peek without re-cloning (supplement; does NOT satisfy the mandatory pull)

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
         │        │        └── quality audit → 18-Check Completion Gate (Bible §09)
         │        │                            + Pristine Sweep (Bible §09 §3)
         │        │
         │        └── existing → Operational Playbook below, ENTER at Phase 0 pre-flight
         │                      (upgrade path; consult Bible §08)
         │                      MANDATORY Phase 1.5 Upstream Capability Sync
         │
         └── new → Operational Playbook, ENTER at Phase 0-NEW (net-new scaffold;
                   consult Bible §03 server standard + §07 contract + §04 skill),
                   then converge at Phase 1.5. Pairing Contract authored in <pair-repo>/docs/
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

### Placeholder & naming legend (resolve EVERY command against this — Law 2 Zero Ambiguity)

The estate distinguishes the bare pair slug from the `-mcp`-suffixed server name. Substitute exactly (worked example: slug `fivetran`):

| Placeholder | Means | Worked example (`<slug>`=fivetran) |
|---|---|---|
| `<slug>` / `<pair>` | bare pair slug (same value; `<pair>` used in code identifiers) | `fivetran` |
| Pair repo / compose dir | `/opt/mcp-servers/<slug>-mcp` | `/opt/mcp-servers/fivetran-mcp` |
| Compose service **and** `container_name` | `<slug>-mcp` | `fivetran-mcp` |
| Image | `<slug>-mcp-<slug>-mcp:<tag>` (compose default `<projectdir>-<service>`) | `fivetran-mcp-fivetran-mcp:v19.1.2` |
| Subdomain | `<slug>-mcp.arjtech.in` | `fivetran-mcp.arjtech.in` |
| Skill dir | `/root/.claude/skills/<slug>` (bare — NOT `-mcp`) | `/root/.claude/skills/fivetran` |
| Source package | `src/<slug>_mcp/` (underscore) | `src/fivetran_mcp/` |
| `<NEW>` / `<CURRENT>` | the SERVER version as a BARE number -- NO leading `v` (commands prepend it: `v<NEW>`, `v<CURRENT>-pre-upgrade`); R13-independent of the framework scale | `19.1.2` |
| `<FRAMEWORK_NEW>` | the Bible framework version only (R13) — **already includes the `Bible-v` prefix; pass it bare in commands, never re-prefix** | `Bible-v19.1.12` |

**Always confirm the real names with `docker ps` / `ls /opt/mcp-servers/` before running estate-mutating commands** — a placeholder mismatch runs against the wrong or a nonexistent target.

---

### PHASE 0-NEW — Net-New Pair Bootstrap (MANDATORY for from-scratch builds ONLY; skip for upgrades)

> Use this phase **instead of** the upgrade Phase 0 when there is no live pair yet. It creates the artifacts the upgrade path assumes already exist, then converges into the shared pipeline at Phase 1.5. (For upgrades, skip directly to Phase 0 below.)

**STEP 0N.1** — Pair-slug selection + scope
- Action: choose `<slug>` (lowercase, matches platform); confirm no existing `/opt/mcp-servers/<slug>-mcp` or `/root/.claude/skills/<slug>` collision
- Action (init candidate-workspace vars ONCE for the whole net-new session — every later step reuses these; NEVER recompute `$(date +%F)`, which a midnight-crossing run would orphan): `export UPGRADE_DIR=/opt/mcp-servers/<slug>-mcp; export BASELINE_DIR=/var/lib/mcp-upgrade-state/<slug>-$(date +%F); mkdir -p /var/lib/mcp-upgrade-state && { mkdir "$BASELINE_DIR" || { echo 'FAIL: run-state dir $BASELINE_DIR already exists (same-day retry or stale state) — rm it or use a fresh run id; reusing a dir merges stale rollback/skill state'; exit 1; }; } && printf net-new > "$BASELINE_DIR/mode"` (DURABLE state dir — NOT `/tmp` — plus an EXPLICIT `net-new` mode marker so Phase 9.3 rollback never has to infer the path from a file's survival)
- VERIFY: local AND remote absence — `test ! -e /opt/mcp-servers/<slug>-mcp && test ! -e /root/.claude/skills/<slug>` AND a 3-way remote check that NEVER fail-opens: `GHV=$(gh repo view ARJ999/<slug>-mcp 2>&1); rc=$?; if [ $rc -eq 0 ]; then echo 'FAIL: remote repo ARJ999/<slug>-mcp already exists'; exit 1; elif printf '%s' "$GHV" | grep -qiE 'could not resolve to a repository|http 404|404: not found'; then :; else echo 'FAIL: cannot VERIFY remote repo (auth/network/API error) — resolve before creating; do NOT fail-open into a repo create'; exit 1; fi` (0 exit = slug taken; explicit not-found = clear to create; anything else = STOP)
- GATE: slug free (local + remote) + `$UPGRADE_DIR`/`$BASELINE_DIR` set

**STEP 0N.2** — Create the pair repo + scaffold the SERVER (MCP repo) and the SKILL (global skills dir) — two distinct locations per R11
- CONSULT: Bible `03_MCP_SERVER_STANDARD.md` (server skeleton: transport, Dockerfile, compose, health) + `07_PAIRING_CONTRACT.md` (contract template) + `04_SKILL_STANDARD.md` (skill scaffold)
- Action (server repo): create then clone into the server path (two steps — `gh repo create --clone` clones into `./<repo>` in cwd, NOT an arbitrary path): `gh repo create ARJ999/<slug>-mcp --private` then `git clone git@github.com:ARJ999/<slug>-mcp.git /opt/mcp-servers/<slug>-mcp`. STOP-IF-FAIL: if the clone or any later 0N scaffold step fails AFTER `gh repo create` succeeded, delete the now-empty remote before retrying so the slug stays free — `gh repo delete ARJ999/<slug>-mcp --yes` (requires confirm/permission)
- Action (server scaffold): under `/opt/mcp-servers/<slug>-mcp/`, author Dockerfile + docker-compose.yml (mount `contract.yaml` as runtime config, NOT baked — see Phase 8.3) + `src/<pkg>/server.py` per Bible §03 (FastMCP, `stateless_http=True`, install_annotator stub wired BEFORE register_all_tools) + `.env.example` per §03 §10 + a `.gitignore` (MUST list `.env`, `.serena/`) + a `.dockerignore` (MUST list `.env`) so secrets can never be committed or baked into the image + a `docker-compose.local.yml` override (DISABLES Traefik router/host labels + binds `127.0.0.1` only) used ONLY for the Phase 0N.4 pre-route LOCAL smoke so a DRAFT candidate is never publicly exposed
- Action (contract — TWO files): author `docs/pairing-contract.md` from `templates/pairing-contract-template.md` (the Markdown doc) AND author `contract.yaml` as real YAML mirroring it (the machine file the enforcer reads), both with `pair_status: DRAFT`. Do NOT copy the .md into the .yaml (Markdown ≠ YAML)
- Action (skill scaffold — STAGED, single-path, R11): `mkdir -p "$BASELINE_DIR/skill-staging"`; copy Bible `templates/skill-template/SKILL.md` → `$BASELINE_DIR/skill-staging/SKILL.md` (NOT the live global path — net-new follows the SAME single-path skill model as upgrades; Phase 9.2 places it live as the fresh first placement). The skill lives ONLY in the global skills dir (placed at Phase 9.2), NEVER inside the MCP repo. (Skill Mesh placement per setup-curator case 9b before first invocation.)
- VERIFY: `python3 -m py_compile /opt/mcp-servers/<slug>-mcp/src/<pkg>/server.py && echo OK`; `test -f "$BASELINE_DIR/skill-staging/SKILL.md" && echo skill-staged`; `test ! -d /opt/mcp-servers/<slug>-mcp/skill && echo r11-clean`
- GATE: server repo cloned + scaffolded (pair_status: DRAFT); skill scaffold present at `$BASELINE_DIR/skill-staging` (NOT live); R11 clean (no `skill/` subdir in MCP repo)

**STEP 0N.3** — Create `.env` (the FIRST `.env`; becomes the sacrosanct baseline)
- Action: author `.env` from `.env.example` (Bible §03 §10) with real credentials set by AJ directly (never pasted in chat — see secure-credential-handoff)
- Action (reuse the `$BASELINE_DIR` set ONCE in STEP 0N.1 — do NOT recompute the date; PATH-EXPLICIT so it works regardless of the operator's cwd): `sha256sum "$UPGRADE_DIR/.env" > "$BASELINE_DIR/env-hash.txt"`
- VERIFY: env-hash.txt non-empty
- GATE: `.env` exists + baseline hash captured (this REPLACES upgrade STEP 0.2 for new builds)

**STEP 0N.4** — First build + LOCAL/container smoke (no rollback image / backup branch yet — nothing to roll back to)
- Action (LOCAL-only smoke — the candidate is DRAFT and must NEVER be publicly routable yet, so run it under a DISTINCT `-p <slug>-mcp-local` project with the `docker-compose.local.yml` override that DISABLES Traefik labels + binds `127.0.0.1` only; `--project-directory "$UPGRADE_DIR"` pins the dir so a copy-paste can't target the wrong project): `IMAGE_TAG=v0.1.0 docker compose -p <slug>-mcp-local --project-directory "$UPGRADE_DIR" -f docker-compose.yml -f docker-compose.local.yml build <slug>-mcp || { echo 'FAIL: local image build failed — refuse to smoke a stale/previous image'; exit 1; }; CFG=$(IMAGE_TAG=v0.1.0 docker compose -p <slug>-mcp-local --project-directory "$UPGRADE_DIR" -f docker-compose.yml -f docker-compose.local.yml config) || { echo 'FAIL: local config render failed'; exit 1; }; echo "$CFG" | grep -qi 'traefik' && { echo 'FAIL: rendered local config STILL carries Traefik labels — the override did not strip them; refuse to start a publicly-routable DRAFT (this gate runs BEFORE up -d)'; exit 1; }; [ "$(IMAGE_TAG=v0.1.0 docker compose -p <slug>-mcp-local --project-directory "$UPGRADE_DIR" -f docker-compose.yml -f docker-compose.local.yml config --format json | jq '[.services[]?.ports // [] | .[] | select(((.host_ip // "0.0.0.0")|test("^(127\\.0\\.0\\.1|::1)$"))|not)] | length')" = 0 ] || { echo 'FAIL: a published host port is NOT bound to loopback — short-syntax ports default to 0.0.0.0 (public) and carry NO host_ip field a YAML grep would catch; bind every port to 127.0.0.1 in docker-compose.local.yml, or publish none (smoke via docker exec)'; exit 1; }; IMAGE_TAG=v0.1.0 docker compose -p <slug>-mcp-local --project-directory "$UPGRADE_DIR" -f docker-compose.yml -f docker-compose.local.yml up -d <slug>-mcp`
- Action (GUARD — prove the DRAFT candidate is NOT publicly exposed; FAIL-CLOSED — the container MUST exist and carry ZERO routing surface): `docker inspect <slug>-mcp >/dev/null 2>&1 || { echo 'FAIL: local smoke container <slug>-mcp not found — cannot verify non-exposure, fail closed'; exit 1; }; TLBL=$(docker inspect <slug>-mcp --format '{{range $k,$v := .Config.Labels}}{{$k}} {{end}}' 2>/dev/null | tr ' ' '\n' | grep -i '^traefik' || true); test -z "$TLBL" || { echo "FAIL: DRAFT local container carries Traefik labels ($TLBL) — the docker-compose.local.yml override must strip ALL traefik.* labels (router/service/host, not just traefik.enable)"; exit 1; }; NLP=$(docker inspect <slug>-mcp --format '{{range $pp,$bb := .NetworkSettings.Ports}}{{range $bb}}{{.HostIp}} {{end}}{{end}}' 2>/dev/null | tr ' ' '\n' | grep -vE '^(127\.0\.0\.1|::1|)$' || true); test -z "$NLP" || { echo "FAIL: DRAFT local container publishes NON-loopback host ports (bound to: $NLP) — bind 127.0.0.1 only"; exit 1; }`. SECONDARY (confirmation only — a DNS/TLS/network error could ALSO make a curl fail, so NOT sufficient alone): `curl -fsS -o /dev/null https://<slug>-mcp.arjtech.in/health && { echo 'FAIL: public subdomain unexpectedly served a DRAFT candidate'; exit 1; } || true`
- VERIFY (container-direct — Traefik/DNS routing does NOT exist yet, so do NOT hit the public subdomain): probe the mapped host port or exec inside the container, e.g. `docker exec <slug>-mcp curl -fsS -X POST http://localhost:8000/mcp -H 'Content-Type: application/json' -H 'Accept: application/json, text/event-stream' -H 'MCP-Protocol-Version: 2025-11-25' -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}' | sed -n 's/^data: //p'` returns a (possibly small) tool list; `docker inspect <slug>-mcp --format='{{.State.Health.Status}}'` healthy. (Public `https://<slug>-mcp.arjtech.in` is validated LATER, after Phase 6 invocation/route wiring.)
- GATE: container serves MCP locally (container-direct)
- Action (clean up the local smoke before converging — the real build/deploy is Phase 7/8/9; leaving the `<slug>-mcp-local` container up risks a later container-name collision): `IMAGE_TAG=v0.1.0 docker compose -p <slug>-mcp-local --project-directory "$UPGRADE_DIR" -f docker-compose.yml -f docker-compose.local.yml down --remove-orphans`
- Note: `$UPGRADE_DIR` (= the new repo) and `$BASELINE_DIR` were set ONCE in STEP 0N.1 and persist for the session; Phase 2+ use `$UPGRADE_DIR/contract.yaml`, `$BASELINE_DIR/...`. A net-new build has no live mount to protect, so it edits the new repo directly.
- → CONVERGE: proceed to **Phase 1.5** (Upstream Capability Sync). From Phase 2 onward the pipeline is identical to the upgrade path (Phase 0.5 Census applies once tools exist); skill handling (Phase 5) stages to `$BASELINE_DIR/skill-staging` like the upgrade path; since a net-new pair has NO prior live skill, the Phase 9.2 apply is a fresh first placement (no rollback baseline needed) — do NOT author the skill at the live global path early.

---

### PHASE 0 — Pre-Flight (UPGRADE path — MANDATORY before any upgrade work; for net-new builds use Phase 0-NEW above)

**STEP 0.1** — Identify the pair and current state
- Action: `cd /opt/mcp-servers/<slug>-mcp && cat pyproject.toml | grep version`
- Action: `docker ps --filter name=<slug>-mcp --format '{{.Image}} {{.Status}}'`
- VERIFY: pair exists, container is healthy, current version captured
- GATE: pair is operational pre-upgrade
- STOP-IF-FAIL: file an issue, do not proceed

**STEP 0.2** — Capture .env baseline (`.env` is sacrosanct — non-negotiable)
- CONSULT: Bible `08_UPGRADE_PLAYBOOK.md § Pre-flight Rule 0` (full rationale + safety clause)
- Action (capture ONE DURABLE baseline dir for the whole upgrade session — every later step reuses `$BASELINE_DIR`, never a `*` glob): `export BASELINE_DIR=/var/lib/mcp-upgrade-state/<slug>-$(date +%F); mkdir -p /var/lib/mcp-upgrade-state && { mkdir "$BASELINE_DIR" || { echo 'FAIL: run-state dir $BASELINE_DIR already exists (same-day retry or stale state) — rm it or use a fresh run id; reusing a dir merges stale rollback/skill state'; exit 1; }; } && printf upgrade > "$BASELINE_DIR/mode" && sha256sum /opt/mcp-servers/<slug>-mcp/.env > "$BASELINE_DIR/env-hash.txt"` — **NOT `/tmp`**: the rollback bundle must survive the documented multi-day (Tier-2 7-day) soak, and a `/tmp` cleanup mid-soak would erase `pre-upgrade-commit.txt` and make a UPGRADE rollback misclassify as NET-NEW (which `rm -rf`s the live pair). The `mode` marker makes UPGRADE-vs-NET-NEW EXPLICIT, never inferred from a file's survival. (Pruned at Phase 10 closure after sign-off.)
- Note: keep `$BASELINE_DIR` exported (or re-export the SAME path) in every shell you run subsequent steps in — this is the single source for env-hash + tool-list + shipped-image-id artifacts
- VERIFY: hash file exists with non-empty content
- GATE: env-hash.txt captured
- STOP-IF-FAIL: never proceed without baseline — rollback safety depends on this

**STEP 0.3** — Snapshot live tool surface
- Action: `curl -fsS -X POST https://<slug>-mcp.arjtech.in/mcp -H 'Content-Type: application/json' -H 'Accept: application/json, text/event-stream' -H 'MCP-Protocol-Version: 2025-11-25' -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}' | sed -n 's/^data: //p' > $BASELINE_DIR/tools-list-pre.json`
- VERIFY: `jq '.result.tools | length' $BASELINE_DIR/tools-list-pre.json` returns the expected baseline count
- GATE: tool inventory snapshotted
- STOP-IF-FAIL: live MCP isn't responding — fix that first

**STEP 0.4** — Tag rollback image + backup branch + ISOLATED upgrade worktree
- Action (tag the EXACT image the live container is running — NOT `:latest`, which may be stale or absent once versioned tags ship): `export RUNNING_IMG=$(docker inspect <slug>-mcp --format '{{.Image}}'); docker tag "$RUNNING_IMG" <slug>-mcp-<slug>-mcp:v<CURRENT>-pre-upgrade`
- Action (PREFLIGHT, then record): the live repo MUST be clean before we pin a rollback point — Phase 9.3 rolls back with `git reset --hard`, which would DESTROY any uncommitted/untracked operator changes, so fail closed rather than silently clobber: `test -z "$(git -C /opt/mcp-servers/<slug>-mcp status --porcelain=v1 --untracked-files=all)" || { echo 'FAIL: live repo has uncommitted/untracked changes — commit/stash/bundle them BEFORE upgrading; a reset --hard rollback would destroy them'; exit 1; }`. THEN record the pre-upgrade live SOURCE state so a post-cutover rollback restores source+contract, not just the image: `export PRE_UPGRADE_COMMIT=$(git -C /opt/mcp-servers/<slug>-mcp rev-parse HEAD); echo "$PRE_UPGRADE_COMMIT" > "$BASELINE_DIR/pre-upgrade-commit.txt"`
- Action (backup branch WITHOUT switching the live working tree): `BK=backup/v<CURRENT>-<slug>-$(date +%F); git -C /opt/mcp-servers/<slug>-mcp branch "$BK" && git -C /opt/mcp-servers/<slug>-mcp push origin "$BK"` (name computed ONCE so a midnight crossing can't create one branch and push another)
- Action (isolated worktree — ALL Phase 2-7 edits happen here, NEVER on the live mount): `export UPGRADE_DIR=/opt/mcp-servers/<slug>-mcp-upgrade; export UPGRADE_BRANCH=upgrade/v<NEW>-<slug>-$(date +%F); git -C /opt/mcp-servers/<slug>-mcp worktree add "$UPGRADE_DIR" -b "$UPGRADE_BRANCH"` — keep `$UPGRADE_BRANCH` exported all session; the Phase 9.2 cutover merge reuses it so a multi-day (7-day Tier-2) canary can never target a date-shifted, non-existent branch name
- **INVARIANT**: the live dir `/opt/mcp-servers/<slug>-mcp/` -- including its **mounted, SIGNED** `contract.yaml` -- is NEVER edited during the upgrade. Phase 2-7 run in `$UPGRADE_DIR`. The live pair keeps reporting SIGNED through the WHOLE upgrade; the candidate is verified AND signed on STAGING (Phase 8) before Phase 9 promotes it, so PROD transitions old-SIGNED -> new-SIGNED directly and is NEVER exposed as DRAFT.
- VERIFY: the pre-upgrade tag resolves to the SAME image the live container runs — `[[ "$(docker inspect <slug>-mcp-<slug>-mcp:v<CURRENT>-pre-upgrade --format '{{.Id}}')" == "$RUNNING_IMG" ]]` AND `test -d "$UPGRADE_DIR"`
- GATE: rollback path exists + isolated worktree created
- STOP-IF-FAIL: never start an upgrade without the rollback tag + worktree (mutating the live contract is banned)

---

### PHASE 0.5 — Tool Health Census (MANDATORY)

**STEP 0.5.1** — Author the Census harness with realistic-args templates
- CONSULT: Bible `14_*.md § R39 Realistic-Args Census Probing` (4-layer probe spec)
- CONSULT: Bible `14_*.md § R38 Upstream Verification Mandate (parent R30)` (banned pattern-inferred guesses)
- Action: write `scripts/tool_health_census.py` per the reference implementation in any signed pair (snowflake-mcp v22, fivetran-mcp v19 are working examples — but the harness itself is pair-agnostic)
- The harness MUST include all 4 probe layers per R39: empty-args probe, realistic-args probe with per-family templates, response-shape probe (catches R40-fixable errors), server-stability probe (rate-limit floor + post-burst /health restart-test).
- VERIFY: `python3 -m py_compile scripts/tool_health_census.py && echo OK`
- GATE: harness compiles + has all 4 probe layers per R39 spec

**STEP 0.5.2** — Run Census and triage
- Action: `python3 scripts/tool_health_census.py --rate-limit-sleep 0.05 2>&1 | tee docs/census-pre-upgrade-$(date +%F).log`
- VERIFY: census completes without exhausting server (`/health` still returns healthy after run)
- GATE: census output produces CSV with outcome classification per tool
- STOP-IF-FAIL: if server returns 404 cascade mid-census, restart server, lower rate, restart census from scratch

**STEP 0.5.3** — Classify findings → action plan
- CONSULT: Bible `14_*.md § R39` outcome rubric (PASS / BROKEN / FEATURE_NOT_ENABLED / probe-gap)
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

**STEP 1.5.2** — VERIFY EVERY MUST-ADOPT BEFORE PHASE 4 (R38)
- CONSULT: Bible `14_*.md § R38 Upstream Verification Mandate (parent R30)` + R38 § Cascade per-tool VERIFY GATE (Check 7.1)
- For each must-adopt item that involves new SQL/REST/CLI calls:
  - Either: live-probe the function/endpoint exists (`SELECT function_name FROM <PLATFORM>.INFORMATION_SCHEMA.FUNCTIONS WHERE function_name = '<NAME>'`)
  - Or: cite the vendor docs URL of the function signature in the upstream-sync proposal
- VERIFY: every must-adopt has either a live-probe receipt or a vendor docs URL
- GATE: zero unverified must-adopts
- STOP-IF-FAIL: a must-adopt without verification is an R38 violation — DO NOT add it to capability_adds_*.py

---

### PHASE 2 — Pairing Contract (MANDATORY — Bible Law 8)

**STEP 2.1** — Draft the 12-section contract.yaml + reset the candidate to DRAFT
- CONSULT: Bible `07_PAIRING_CONTRACT.md` (12-section spec + Core 8 + Domain-Conditional 2)
- TWO distinct artifacts (a pair carries BOTH — see fivetran-mcp): (a) `docs/pairing-contract.md` = the human-authoritative Markdown contract, authored from `templates/pairing-contract-template.md`; (b) `contract.yaml` = the machine-consumable YAML the Contract Enforcer reads at startup, authored/regenerated to mirror the .md fields. Do NOT copy the .md into contract.yaml (Markdown is not YAML — `yaml.safe_load` on the template raises a scanner error).
- Action (doc): fill all 12 sections of `docs/pairing-contract.md` from the .md template (§6 is empty per G2 removal in v19.1.2)
- Action (machine): author/regenerate `contract.yaml` as real YAML mirroring the doc — canonical Bible §07 fields: `contract_version` (schema, fixed), `pair_slug`, `pair_status`, `skill_version`, `mcp_image_tag` (`<repo>:<tag>`), `framework_version`, `tool_inventory`, `banned_tools`, `guardrails`, …
- Action (in `$UPGRADE_DIR` ONLY — never the live mount): set the CANDIDATE status to DRAFT on BOTH mirror surfaces (R11 lockstep — machine `contract.yaml` + human-authoritative `docs/pairing-contract.md`): set the canonical SINGLE status value (precise whole-line SET, not a broad word-sed): `sed -i 's/^pair_status:.*/pair_status: DRAFT/' "$UPGRADE_DIR/contract.yaml"; sed -i -E 's/^\*\*(Pair status|Status)\*\*:.*/**Pair status**: DRAFT/' "$UPGRADE_DIR/docs/pairing-contract.md" && [ "$(grep -cE '^\*\*Pair status\*\*: DRAFT$' "$UPGRADE_DIR/docs/pairing-contract.md")" = 1 ] || { echo 'FAIL: human contract not normalized to EXACTLY ONE canonical **Pair status**: DRAFT line — a pair may use a non-canonical header (a **Status**: form); the alternation normalizes it and the count assertion fails closed rather than silently leaving the human mirror stale'; exit 1; }`. The LIVE production contract files (mounted, SIGNED) are untouched and keep reporting SIGNED. (New builds scaffold the candidate as DRAFT already.) The candidate stays DRAFT through build (Phase 7) and staging deploy (Phase 8.1); **STEP 8.3** is the single point that flips it to SIGNED — AFTER Phase 8.2 staging verify (Bible §09:208) and BEFORE the Phase 8.4 final 18-check gate (so Check 3 "Pairing Contract signed" is GREEN). Phase 9 then promotes the already-signed candidate.
- VERIFY: `python3 -c "import yaml; yaml.safe_load(open('$UPGRADE_DIR/contract.yaml'))"` passes (real YAML) AND `grep '^pair_status:' "$UPGRADE_DIR/contract.yaml"` returns DRAFT AND `test -f "$UPGRADE_DIR/docs/pairing-contract.md"`
- GATE: contract.yaml is valid YAML + docs/pairing-contract.md present + all sections populated + pair_status: DRAFT

**STEP 2.2** — Declare deployment_tier per R16
- CONSULT: Bible `06_STACK_MANIFEST.md § 1.6.1` Supply-Chain Tier Gating R16
- Action: choose Tier 1 / 2 / 3 with rationale (Tier 1 = personal VPS R&D; Tier 2 = single-org production; Tier 3 = multi-tenant enterprise)
- VERIFY: `grep '^deployment_tier:' "$UPGRADE_DIR/contract.yaml"` returns 1/2/3
- GATE: tier declared with rationale
- STOP-IF-FAIL: never gold-plate (Tier 3 reqs on Tier 1 pair) — banned

**STEP 2.3** — Banned-tool list for Gen 1 / deprecated tools
- CONSULT: Bible `07_PAIRING_CONTRACT.md § 3` (banned-tool fields spec)
- For each tool to be banned, the Bible §07 canonical fields: `name` + `reason` + `replacement` (a tool-name or `none`) + `ban_scope` (`always` or `context:<condition>`) + `ban_since_version`
- VERIFY (YAML-aware — iterate every list item, do NOT grep): `python3 -c "import yaml,sys; b=yaml.safe_load(open('$UPGRADE_DIR/contract.yaml')).get('banned_tools',[]); req={'name','reason','replacement','ban_scope','ban_since_version'}; bad=[t.get('name','?') for t in b if not req.issubset(t)]; sys.exit('MISSING fields on: '+','.join(bad)) if bad else print(f'OK {len(b)} banned tools, all 5 Bible fields')"`
- GATE: every banned tool has all 5 Bible §07 fields (name, reason, replacement, ban_scope, ban_since_version) per the YAML-aware check

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
- VERIFY (STATIC candidate check, PRE-deploy — R13 / Bible §09 Check 10): encode the dir in the command (the script reads compose/contract from CWD): `cd "$UPGRADE_DIR" && bash /root/aj-workspace/scripts/cascade-check.sh --static v<NEW> <FRAMEWORK_NEW> <slug>` exits 0. `--static` checks only file surfaces (compose labels, contract.yaml framework_version, src no-hardcode) and does NOT curl live `/health` — during Phase 3 the live pair is still the OLD version, so a live compare would false-fail every valid upgrade. `<NEW>` = SERVER version; `<FRAMEWORK_NEW>` = INDEPENDENT Bible version (already carries `Bible-v`; a server bump does NOT imply a framework bump). The LIVE cascade gate (full, incl /health) runs POST-promotion in Phase 9.3. Exit 1 = drift — fix it, never proceed.
- GATE: `cascade-check.sh` exit 0 — FAIL-CLOSED: it requires AND matches ≥1 server-scale surface AND ≥1 framework-scale surface (a scale with zero present surfaces FAILS; every PRESENT surface must match). It does NOT require every possible surface to exist, so still hand-bump every surface in the Action above; the script proves no DRIFT on the surfaces that exist.

**STEP 3.3** — .env hash check post-bump
- Action (`.env` is sacrosanct and lives ONLY at the live canonical path — it is never copied into the worktree): `NEW_HASH=$(sha256sum /opt/mcp-servers/<slug>-mcp/.env | awk '{print $1}'); BASE=$(cat $BASELINE_DIR/env-hash.txt | awk '{print $1}'); [[ "$NEW_HASH" == "$BASE" ]] || { echo "FAIL: .env changed since baseline"; exit 1; }`
- GATE: .env unchanged
- STOP-IF-FAIL: investigate immediately — never proceed if .env touched

---

### PHASE 4 — Server-side Enforcement + Capability Adds (BIG PHASE — most discipline required)

**STEP 4.1** — install_annotator wired BEFORE register_all_tools
- CONSULT: Bible `03_MCP_SERVER_STANDARD.md § install_annotator pattern` (with R24 backend gate + R40 result-shape wrap)
- Action: server.py `install_annotator(mcp)` MUST be called BEFORE any tool registration
- VERIFY (assert ordering, do not eyeball): `python3 -c "import re,sys; s=open('src/<pkg>/server.py').read(); a=s.find('install_annotator(mcp)'); r=s.find('register_all_tools'); sys.exit('install_annotator missing' if a<0 else 'register_all_tools missing' if r<0 else 'ORDER WRONG: annotator must precede registration' if a>r else print('OK annotator precedes registration'))"`
- GATE: install_annotator wraps mcp.tool before any registration call

**STEP 4.2** — install_annotator includes R40 result-shape wrapping
- CONSULT: Bible `14_*.md § R40` Result-Shape Wrapping (envelope spec + reference handler)
- Action: enforced_handler must wrap non-dict returns into `{"status":"success","data": ...}` envelope per R40 spec
- VERIFY: `grep -A5 'def _wrap' src/<pkg>/core/tool_annotations.py` shows the wrapper logic
- GATE: R40 wrap function present in install_annotator
- STOP-IF-FAIL: without R40, FastMCP throws ValueError on non-dict returns and breaks tool packaging

**STEP 4.3** — Contract Enforcer wired (G6 banned-tool block)
- CONSULT: Bible `07_PAIRING_CONTRACT.md § 5 G6` (banned-tool dispatch behavior + -32006 spec)
- Action: server.py `from .core.contract_enforcer import get_enforcer; _enforcer = get_enforcer()` at module init
- VERIFY: probe EVERY tool in contract `banned_tools` with `{}` args; each returns JSON-RPC `-32006` + migration message
- GATE: every contract-declared banned tool (N per `banned_tools`, not a fixed count) returns -32006

**STEP 4.4** — Capability adds MUST pass R38 upstream-verification per tool
- CONSULT: Bible `14_*.md § R38 Upstream Verification Mandate (parent R30)` + R38 § Cascade per-tool VERIFY GATE (Check 7.1)
- For each new tool in capability_adds_*.py:
  - STEP 4.4.a — write the tool implementation
  - STEP 4.4.b — invoke it against live target with realistic args (or cite vendor docs URL in source comment)
  - STEP 4.4.c — record `verified_at: <date>` in contract.yaml `tool_inventory` per-tool list
- STOP-IF-FAIL: if a tool's underlying SQL function / REST endpoint can't be verified, DO NOT register it. Pattern-inferred guesses (e.g., `SHOW <FEATURE> <NOUN>` extrapolated from another feature) are BANNED per R38 (Upstream Verification Mandate).
- GATE: every tool in capability_adds_*.py has either verified_at timestamp OR vendor docs URL in source comment

**STEP 4.5** — G8 redact_processor in structlog chain
- CONSULT: Bible `07_PAIRING_CONTRACT.md § 5 G8` (redact patterns + processor chain spec)
- Action: insert `g8_redact_processor` in structlog.configure processor chain BEFORE `JSONRenderer`
- VERIFY: log a test event with a credential-named field (e.g., `<vendor>_private_key`, `api_key`, `password`); assert log line shows `***REDACTED***`
- GATE: redact_processor verified active

**STEP 4.6** — Register the APPLICABLE MCP primitives via `.register(mcp)` (Check 12.4)
- CONSULT: Bible `03_MCP_SERVER_STANDARD.md § 6` (mandate table) + `§ 6.1` (registration pattern)
- Mandate per Bible §6 (NOT all unconditional): Resources — MUST if the pair exposes read-only data views; Prompts — MUST if it ships canonical prompt templates; Elicitation — MUST **if** the pair has interactive bootstrap flows (OAuth consent / confirmation / secret entry); Sampling — **MAY** (only for pairs that delegate LLM calls back to the client)
- Action: for each primitive that APPLIES to this pair, server.py instantiates AND calls `.register(mcp)` (banned pattern: instantiate without register — the class exists but the agent can't invoke it)
- VERIFY: container startup log / introspection shows each *applicable* primitive registered; do NOT force elicitation/sampling onto a pair that has no interactive flow / no client-sampling need
- GATE: every applicable primitive registered via `.register(mcp)`; none instantiated-but-unregistered

**STEP 4.7** — Static-arity audit (R45 OR R49 per pair architecture; v19.1.9 / v19.1.11)
- CONSULT: Bible `16_FRAMEWORK_REFINEMENTS_v19.1.9.md` (R45 two-layer) OR `18_FRAMEWORK_REFINEMENTS_v19.1.11.md` (R49 all-in-one-class)
- Action: per pair architecture (declared in contract.yaml § Architecture from STEP 2.4), run the corresponding audit:
  - Two-layer (R45): `python3 /root/aj-workspace/MCP-God-Agent-Development-Bible/framework/scripts/wrapper_arity_audit.py --server-file src/<pair>_mcp/server.py --client-file src/<pair>_mcp/core/api_client.py --client-symbol <pair>_client --client-class <Pair>APIClient` (script EXISTS in the Bible repo)
  - All-in-one-class (R49): `python3 /root/aj-workspace/MCP-God-Agent-Development-Bible/framework/scripts/intra_class_arity_audit.py --class-file src/<pair>_client_complete.py --class-name <Pair>Client` — **NOTE: this script's authoring is still pending in the Bible (R49 ships AST pseudocode only).** Until it lands, perform the intra-class arity audit MANUALLY per the Bible §18 pseudocode (the snowflake-mcp v22.0.5 R49 pilot did exactly this)
- VERIFY: audit (script OR manual) exits clean (or `--soft-warn` first cycle); confirm the wrapper script exists before invoking (Bible-repo asset; `python3 <script>` needs no +x) — `test -f /root/aj-workspace/MCP-God-Agent-Development-Bible/framework/scripts/wrapper_arity_audit.py`
- GATE: 0 issues. `--soft-warn` is a FIRST-ADOPTION-cycle concession ONLY (a legacy pair newly adopting R45/R49) with the backlog tracked in `pilot-learnings.md`; the Phase 8.4 final gate (Check 16/17) is HARD -- a SIGNED pair MUST have a clean static-arity audit.
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

### PHASE 5 — Skill Upgrade (STAGED — the live global skill is NOT touched until Phase 9 promotion)

> The skill is the PAIRED half and MUST cut over ATOMICALLY with the MCP (pair invariant — prod must never run old-MCP + new-skill). So Phase 5 edits a STAGED copy + backs up the live skill; Phase 9.2 applies skill + hooks together with the MCP cutover; Phase 9.3 rollback restores both. A Phase-8 failure leaves the live skill untouched (nothing to roll back).

**STEP 5.0** — Back up the live skill + create the staging copy
- Action: `if [ -d /root/.claude/skills/<slug> ]; then rm -rf "$BASELINE_DIR/skill-pre-upgrade" "$BASELINE_DIR/skill-staging"; cp -a /root/.claude/skills/<slug> "$BASELINE_DIR/skill-pre-upgrade"; mkdir -p "$BASELINE_DIR/skill-staging"; cp -a /root/.claude/skills/<slug>/. "$BASELINE_DIR/skill-staging/"; fi` (REPLACE, never merge — `rm -rf` the targets first so a reused dir can't nest stale skill files) (UPGRADE: back up the live skill + seed staging from it. NET-NEW: `$BASELINE_DIR/skill-staging` was already seeded from the template in STEP 0N.2 and there is NO live skill to back up.) — staging lives in `$BASELINE_DIR` (OUTSIDE the MCP repo, so it is never caught by the Phase 7.1 `git add -A`; R11: no skill content inside the MCP repo). The live skill is the rollback baseline; ALL Phase-5 edits target `$BASELINE_DIR/skill-staging/`, NEVER the live path.
- GATE: staging copy exists (ALL paths); live-skill backup exists (UPGRADE only — net-new has no live skill to back up); live skill untouched.

**STEP 5.1** — Update skill frontmatter (Bible §04 §3) — in the STAGING copy
- CONSULT: Bible `04_SKILL_STANDARD.md § 2`.
- Action (edit `$BASELINE_DIR/skill-staging/SKILL.md`): name, description, version, mcp_server, mcp_tool_prefix, mcp_tool_count, mcp_auth, mcp_endpoint, pairing_contract_version, pairing_contract_path.
- VERIFY (smoke PRESENCE check — counts DISTINCT keys, NOT full YAML validation): `awk 'NR==1&&/^---/{f=1;next} /^---/{f=0} f' "$BASELINE_DIR/skill-staging/SKILL.md" | grep -oE '^(name|description|version|mcp_server|mcp_tool_prefix|mcp_tool_count|mcp_auth|mcp_endpoint|pairing_contract_version|pairing_contract_path):' | sort -u | wc -l` returns 10.
- GATE: 10/10 mandatory keys PRESENT — a smoke presence-check ONLY; authoritative skill validation (values, no duplicate/conflicting keys) is Bible §04 + the Phase 8.4 Check 6 over the staged content.

**STEP 5.2** — Skill body has 9 mandatory sections — in the STAGING copy
- VERIFY (smoke PRESENCE check — counts DISTINCT headings via `sort -u`, so a duplicated heading cannot mask a missing one; ordering/uniqueness are authoritatively validated by Bible §04 + Phase 8.4 Check 6, NOT here): `grep -oE '^## (Identity|Pairing Block|When to Invoke|Tool Map|Pre-Call Sequences|Guardrails|Verification Rituals|Cascade Notes|Gotcha Registry)' "$BASELINE_DIR/skill-staging/SKILL.md" | sort -u | wc -l` returns 9.
- GATE: 9/9 distinct sections present (smoke-only — NOT a structural validator).

**STEP 5.3** — R11 cleanup: no `skill/` subdir inside MCP repo (staged)
- VERIFY: `test ! -e "$UPGRADE_DIR/skill"` (no `skill/` subdir in the MCP repo — R11) OR a documented archive; `test` is `set -e`-safe where a bare `ls` of an absent dir would abort the shell.
- GATE: skill content lives ONLY in the global skills dir. The byte-identical R11 dual-mirror (`/root/.claude/skills/<slug>` <-> `/home/claude/.claude/skills/<slug>`) and live placement happen ATOMICALLY in Phase 9.2 — NOT here.

---

### PHASE 6 — Invocation Mechanism Wiring (STAGED; live hooks applied at Phase 9.2 promotion)

**STEP 6.1** — Prepare the declared invocation layers (STAGE; do NOT mutate live hooks until Phase 9.2)
- CONSULT: Bible `08_UPGRADE_PLAYBOOK.md § Phase 6` (Layers 1, 2, 4) + `09_VERIFICATION_PROTOCOL.md § Check 10` (Layer 3 drift — **AUTHORITATIVE**).
- Layer 1 (always): the STAGED skill frontmatter fields resolve (event-driven drift picks them up post-promotion).
- Layer 2 (if declared): back up the live registry — `cp -a /root/.claude/hooks/skill-autoloader-registry.json "$BASELINE_DIR/skill-autoloader-registry.json.pre"` — AND MATERIALIZE the full post-cutover registry (live content + the new skill↔MCP mapping entry) as a CONCRETE file `$BASELINE_DIR/skill-autoloader-registry.json.staged`; APPLY it (copy over the live file) only in Phase 9.2 (so prod never carries new autoload metadata for an un-promoted MCP). The PRESENCE of `.staged` is the per-pair signal that Layer 2 is in play — pairs with NO Layer 2 stage neither `.pre` nor `.staged`, and Phase 9.2/9.3 skip the registry surface accordingly.
- Layer 3 (always): confirm `scripts/drift-check.sh` is INVOCABLE + wired to run **post every `docker compose up -d`** (event-driven, R14). Do NOT install a nightly cron (Bible `09 § Check 10`; supersedes the stale §08 nightly-cron text — flag for Bible cascade in Phase 10.5).
- Layer 4 (if declared): `X-Claude-Skill-Loaded` header validator in server middleware (ships inside the candidate image).
- VERIFY (POST-promotion, in Phase 9.3): with the skill applied, calling any `mcp__<prefix>__*` WITHOUT invoking the skill → Layer-2 block or Layer-3 alert; invoke skill, retry → succeeds.
- GATE: layers STAGED; live hooks unchanged until Phase 9.2. STOP-IF-FAIL (post-promotion): Layer-2 blocking legitimate Inspector calls → `CLAUDE_SKIP_SKILL_AUTOLOAD=1` bypass for dev context (Bible §08 Phase 6 rollback)

---

### PHASE 7 — Build the Release Artifact (single build — verified == shipped)

> **Reproducibility invariant (Law 12)**: the image built here (identified by its local `.Id`) is the EXACT image that gets verified on STAGING (Phase 8.2 pre-sign + Phase 8.4 final gate) and shipped (Phase 9 promotion). There is NO rebuild anywhere downstream. Signing (`pair_status: SIGNED`, Phase 8.3 on STAGING) flips the **mounted** candidate contract + `docker compose restart` (config reload) — the image `.Id` is never re-created. The OLD live contract stays SIGNED until Phase 9.2 promotes the already-signed candidate; the upgrade worktree (`$UPGRADE_DIR`), not the live mount, holds the candidate until then. Downstream phases PROMOTE this image ID; they never rebuild it.

**STEP 7.1** — Build the artifact ONCE from DRAFT source + record its local image ID
- Precondition: `grep '^pair_status:' "$UPGRADE_DIR/contract.yaml"` returns **DRAFT** (set in Phase 2; the candidate is NOT signed at build time — signing happens on STAGING in Phase 8.3, only after Phase 8.2 staging verify (workhorse+census) passes, Bible §09:208)
- Action (COMMIT the candidate first — the built image and the Phase 9.2 cutover merge MUST reference the SAME source, for reproducibility + honest rollback/audit): worktree clean of stray files; PRE-COMMIT GATES (fail-closed): `git -C "$UPGRADE_DIR" check-ignore .env >/dev/null 2>&1 && ! git -C "$UPGRADE_DIR" ls-files --error-unmatch .env >/dev/null 2>&1 || { echo 'FAIL: .env must be gitignored AND untracked (check-ignore passes + ls-files shows it is not tracked) — never commit secrets'; exit 1; }` AND `[ ! -e "$UPGRADE_DIR/skill-staging" ] || { echo 'FAIL: skill staging must live OUTSIDE the MCP repo (R11) — use $BASELINE_DIR/skill-staging'; exit 1; }` AND (no untracked NON-allowlisted residue can be swept into the release commit by `git add -A` — an explicit cleanliness gate, not just the credential filename scan): `STRAY=$(git -C "$UPGRADE_DIR" ls-files --others --exclude-standard | grep -vE '^(src/|tests/|docs/|scripts/|pyproject\.toml$|uv\.lock$|poetry\.lock$|requirements[^/]*\.txt$|docker-compose|Dockerfile|contract\.yaml$|\.gitignore$|\.dockerignore$|README|LICENSE|Makefile$|\.env\.example$)' || true); test -z "$STRAY" || { echo "FAIL: untracked files outside the expected source allowlist would be committed by git add -A (likely generated residue): $STRAY — gitignore or remove them first"; exit 1; }` AND (nothing is ALREADY staged before this step — so the candidate commit contains ONLY what this step decides to stage, never a pre-existing staged generated artifact): `git -C "$UPGRADE_DIR" diff --cached --quiet || { echo 'FAIL: pre-existing STAGED changes before the candidate commit — git reset to unstage them first'; exit 1; }`. THEN: `git -C "$UPGRADE_DIR" add -A && { SECRETS=$(git -C "$UPGRADE_DIR" diff --cached --name-only | grep -iE '(^|/)\.env($|\.)|\.pem$|\.key$|\.p12$|\.pfx$|(service[-_]?account|[-_]sa)[^/]*\.json$' | grep -ivE '(^|/)\.env\.(example|sample|template)$' || true); test -z "$SECRETS" || { git -C "$UPGRADE_DIR" reset -q; echo "FAIL: STAGED credential material would be committed: $SECRETS — scanned AFTER \`git add -A\` so UNTRACKED secrets are caught too (not just tracked); gitignore + remove them, then retry"; exit 1; }; } && { if command -v gitleaks >/dev/null 2>&1; then gitleaks protect --staged --no-banner -s "$UPGRADE_DIR" || { git -C "$UPGRADE_DIR" reset -q; echo "FAIL: gitleaks flagged staged SECRET CONTENT (a credential pasted INTO an allowed file — filename scans alone are insufficient)"; exit 1; }; else CHITS=$(git -C "$UPGRADE_DIR" diff --cached -U0 | grep -E '^\+' | grep -nEi 'AKIA[0-9A-Z]{16}|-----BEGIN [A-Z ]*PRIVATE KEY-----|ghp_[0-9A-Za-z]{36}|AIza[0-9A-Za-z_-]{35}|xox[bapsr]-[0-9A-Za-z-]+|(password|secret|token|api[_-]?key)[[:space:]]*[:=][[:space:]]*[A-Za-z0-9/+=_.-]{12,}' || true); test -z "$CHITS" || { git -C "$UPGRADE_DIR" reset -q; echo "FAIL: staged CONTENT matches a credential pattern (fallback scan — install gitleaks for full coverage): $CHITS — never commit secrets, use .env"; exit 1; }; fi } && git -C "$UPGRADE_DIR" commit -m 'candidate: <slug> v<NEW> (pair_status DRAFT)' && export UPGRADE_COMMIT=$(git -C "$UPGRADE_DIR" rev-parse HEAD) && git -C "$UPGRADE_DIR" show "$UPGRADE_COMMIT:contract.yaml" | grep -q '^pair_status: DRAFT' && echo "$UPGRADE_COMMIT" > "$BASELINE_DIR/upgrade-commit.txt"` (the chain is `&&`-joined end to end — a failed `commit` aborts before any SHA is captured or persisted, so a stale/empty commit can never be recorded; the `show ... | grep DRAFT` proves the recorded commit IS the DRAFT candidate that gets built, not an unrelated HEAD). Build ONLY from this committed state; nothing uncommitted enters the image. The commit is persisted to `$BASELINE_DIR/upgrade-commit.txt` so cutover survives a multi-day soak / shell loss.
- Precondition (stable image NAME across build/stage/promote regardless of compose-project/dir): the pair `docker-compose.yml` MUST set an explicit `image: <slug>-mcp-<slug>-mcp:${IMAGE_TAG}` key — a `build:`-only service derives its image name from the project/dir, which would diverge between the worktree build and prod. `[ "$(IMAGE_TAG=v<NEW> docker compose -p <slug>-mcp --project-directory "$UPGRADE_DIR" config --images <slug>-mcp | head -1)" = "<slug>-mcp-<slug>-mcp:v<NEW>" ] || { echo 'FAIL: the <slug>-mcp SERVICE image does not resolve to the EXACT expected <slug>-mcp-<slug>-mcp:v<NEW> — rollback (tagged <slug>-mcp-<slug>-mcp:v<CURRENT>-pre-upgrade) and every deploy assume that precise name; a mere any-image: check would pass a wrong/sidecar image and then fail rollback. Set image: <slug>-mcp-<slug>-mcp:${IMAGE_TAG} on the <slug>-mcp service'; exit 1; }`
- Action (build from a PRISTINE checkout of the recorded commit — NOT the mutable worktree — so the shipped image provably equals `$UPGRADE_COMMIT` and no uncommitted, ignored, or secret file can leak into the build context, Law 12): `BUILD_CTX="$BASELINE_DIR/build-ctx"; rm -rf "$BUILD_CTX" && mkdir -p "$BUILD_CTX" && git -C "$UPGRADE_DIR" archive "$UPGRADE_COMMIT" | tar -x -C "$BUILD_CTX"` (`git archive` emits ONLY tracked files at that exact SHA — no `.env`, no `.serena/`, no post-commit drift), THEN build FROM that context (`-p <slug>-mcp` pins the project so the image name still matches prod): `IMAGE_TAG=v<NEW> docker compose -p <slug>-mcp --project-directory "$BUILD_CTX" build <slug>-mcp`. `$BUILD_CTX` is transactional scratch — `rm -rf "$BUILD_CTX"` once STEP 7.1's image `.Id` is recorded below.
- Action: record the verified-artifact identity by COMPOSE-RESOLVED ref (works whether `image:` is explicit or build-derived) + its `.Id` (the `.Id` is the invariant; the name may vary): `IMG_REF=$(IMAGE_TAG=v<NEW> docker compose -p <slug>-mcp --project-directory "$BASELINE_DIR/build-ctx" config --images <slug>-mcp | head -1) && echo "$IMG_REF" > "$BASELINE_DIR/image-ref.txt" && docker image inspect "$IMG_REF" --format '{{.Id}}' > "$BASELINE_DIR/shipped-image-id.txt" && test -s "$BASELINE_DIR/shipped-image-id.txt" && rm -rf "$BASELINE_DIR/build-ctx"` (every step `&&`-chained, so a failed `image inspect` ABORTS before the build context is deleted; `test -s` proves the `.Id` was actually written) (the `git archive` build context is transactional scratch — removed HERE, now that the `.Id` is recorded; the image itself persists in the Docker store)
- VERIFY: `image-ref.txt` resolves to a real image (`docker image inspect "$(cat "$BASELINE_DIR/image-ref.txt")" --format '{{.Id}}'` non-empty `sha256:...`) == `shipped-image-id.txt`
- GATE: single image built + tagged + image ID recorded (this exact ID is what Phase 8 soaks and Phase 9 promotes — no rebuild in between)

**STEP 7.2** — Tier-appropriate scan (R16)
- CONSULT: Bible `06_STACK_MANIFEST.md § 1.6.1` Supply-Chain Tier Gating
- Tier 1: `trivy image --severity HIGH,CRITICAL --exit-code 1 --output "$UPGRADE_DIR/docs/trivy-scan-$(date +%F).txt" <slug>-mcp-<slug>-mcp:v<NEW>` (`--output` writes the report to the evidence file directly — positive evidence even on a CLEAN scan — while PRESERVING the scan exit code, unlike a `| tee` pipe) — NOT blanket-informational (Bible Check 8 expects clean or DOCUMENTED waiver). HIGH/CRITICAL WITH an available upstream fix MUST be patched; HIGH/CRITICAL OS-dep with NO upstream fix is waivable but MUST be recorded in `docs/trivy-scan-<DATE>.txt` with rationale (then re-run with that CVE in `--ignore` or accept the documented exit). ALWAYS write a scan record `docs/trivy-scan-<DATE>.txt` (even a CLEAN scan: "no HIGH/CRITICAL" — Check 8 wants positive evidence, not silence) and STAGE it so it rides into the promoted source at sign: `git -C "$UPGRADE_DIR" add docs/trivy-scan-*.txt` (committed alongside the contract mirrors in Phase 8.3; the IMAGE was already built from `$UPGRADE_COMMIT`, so this source-only evidence file never alters the verified artifact)
- Tier 2+: trivy + SBOM (syft) + digest-pin
- Tier 3: + cosign + SLSA-L3
- VERIFY: scan output captured to `docs/trivy-scan-<DATE>.txt`
- GATE: tier-appropriate artifacts present
- STOP-IF-FAIL (Tier 2+): if any HIGH with available upstream fix, MUST patch — then rebuild (returns to STEP 7.1; the rebuilt image ID becomes the new verified-artifact identity)

---

### PHASE 8 — Staging: VERIFY -> SIGN -> FINAL GATE (all on the candidate; PROD is NEVER touched here)

> Bible §09:208 + §07 ban cutting PROD over to an unverified OR unsigned candidate. So pre-sign verification, signing, AND the 18-check Completion Gate ALL run HERE, on STAGING, against the exact `v<NEW>` image (`.Id` from Phase 7). PROD keeps running the prior-cycle SIGNED pair, untouched. Phase 9 then PROMOTES the already-verified, already-signed candidate. Any Phase-8 failure -> discard the candidate worktree/branch; PROD needs NO rollback (it was never mutated).

**STEP 8.1** -- Deploy candidate to staging + soak
- Action (INSPECT the rendered staging config BEFORE any deploy — a shared label/container/host could hijack prod routing the instant `up` runs, so the gate must precede deployment): the pair repo carries a staging override `docker-compose.staging.yml` setting a DISTINCT `container_name: <slug>-mcp-staging`, DISTINCT Traefik router+service label names, host rule `<slug>-mcp-staging.arjtech.in`, and the candidate `contract.yaml` (DRAFT) mount. Render: `IMAGE_TAG=v<NEW> docker compose -p <slug>-mcp-staging --project-directory "$UPGRADE_DIR" -f docker-compose.yml -f docker-compose.staging.yml config`.
- GATE (HARD, PRE-DEPLOY — executable comparator, never eyeballed): the set-intersection of prod vs staging rendered names MUST be EMPTY (fail-closed capture, never eyeballed): `prod_cfg=$(docker compose -p <slug>-mcp --project-directory /opt/mcp-servers/<slug>-mcp config) || { echo 'FAIL: prod compose config render failed — cannot prove disjointness, fail closed (do NOT deploy)'; exit 1; }; stag_cfg=$(IMAGE_TAG=v<NEW> docker compose -p <slug>-mcp-staging --project-directory "$UPGRADE_DIR" -f docker-compose.yml -f docker-compose.staging.yml config) || { echo 'FAIL: staging compose config render failed — fail closed'; exit 1; }; NAMEPAT='container_name:.*|traefik\.http\.(routers|services)\.[a-zA-Z0-9_-]+|<slug>-mcp(-staging)?\.arjtech\.in'; prod_names=$(printf '%s' "$prod_cfg" | grep -oE "$NAMEPAT" | sort -u); stag_names=$(printf '%s' "$stag_cfg" | grep -oE "$NAMEPAT" | sort -u); test -n "$prod_names" && test -n "$stag_names" || { echo 'FAIL: the name-extraction regex matched ZERO surfaces on prod and/or staging — cannot prove disjointness on empty sets (a silent NAMEPAT miss would falsely pass); fix the pattern, fail closed'; exit 1; }; collisions=$(comm -12 <(echo "$prod_names") <(echo "$stag_names")); test -z "$collisions" || { echo "FAIL: staging shares container/router/service/host names with prod: $collisions"; exit 1; }; echo "$stag_cfg" | grep -oE 'source: /opt/mcp-servers/[^[:space:]]+' | grep -vF "source: $UPGRADE_DIR" | grep -q . && { echo "FAIL: staging bind-mounts a /opt/mcp-servers path OUTSIDE \$UPGRADE_DIR (likely the LIVE prod dir/contract/data) — staging must mount the candidate from \$UPGRADE_DIR ONLY; fix the override"; exit 1; } || true` (mount disjointness — names alone don't prove volume/config-mount isolation; each `config` render is captured into a var with `|| exit 1` BEFORE the comparison and stderr is NOT suppressed — if either render fails the gate fails closed instead of comparing two empty sets and falsely passing) — any shared name → STOP and fix the override; do NOT deploy.
- Action (only AFTER the pre-deploy gate passes): `IMAGE_TAG=v<NEW> docker compose -p <slug>-mcp-staging --project-directory "$UPGRADE_DIR" -f docker-compose.yml -f docker-compose.staging.yml up -d <slug>-mcp`.
- Soak: Tier 1 = abbreviated smoke; Tier 2+ = 7-day observation (p50/p95/p99 latency, error rate, guardrail fires, cost vs baseline).
- VERIFY: staging `/health` healthy at `<slug>-mcp-staging.arjtech.in`; `/health.contract.pair_status` = DRAFT (honest -- not signed yet); `docker inspect <slug>-mcp-staging --format='{{.Image}}'` == `$(cat "$BASELINE_DIR/shipped-image-id.txt")` (the verified Phase-7 `.Id`).
- GATE: error rate <= baseline; p95 <= 1.1x baseline; zero unexpected guardrail fires.

**STEP 8.2** -- Pre-sign verification ON STAGING (Bible §09:208 -- gates the sign; MUST pass before signing)
- CONSULT: Bible `09_VERIFICATION_PROTOCOL.md § verification_rituals` (workhorse smoke) + `14_*.md § R39` (4-layer census).
- PRECONDITION (prove the surface under test IS the staging artifact, NOT prod/default — else the whole verify-on-staging gate is defeated): `[ "$(docker inspect <slug>-mcp-staging --format '{{.Image}}' 2>/dev/null)" = "$(cat "$BASELINE_DIR/shipped-image-id.txt")" ] || { echo 'FAIL: staging container is NOT running the Phase 7 verified image .Id — refuse to verify the wrong artifact'; exit 1; }` AND `curl -fsS https://<slug>-mcp-staging.arjtech.in/health | jq -e '.version=="v<NEW>"' >/dev/null || { echo 'FAIL: staging /health is not v<NEW> — wrong surface under test'; exit 1; }`. Every probe below targets the STAGING host EXPLICITLY.
- Workhorse smoke (vs staging — target the STAGING host EXPLICITLY): every contract `§10 workhorse_smoke` category tool, invoked against `https://<slug>-mcp-staging.arjtech.in/mcp`, returns `"status":"success"` against live upstream (identity probe + tool-count are NOT a substitute -- §09:210).
- R39 4-layer census (vs staging — `--endpoint` pins the STAGING host so the census probes the verified artifact, NEVER the script's default prod endpoint): `python3 scripts/tool_health_census.py --endpoint https://<slug>-mcp-staging.arjtech.in/mcp --rate-limit-sleep 0.05`; triage every BROKEN row per Phase 0.5.3.
- GATE: workhorse smoke 100% AND census 0 TRUE_BUG (PROBE_GAP / FEATURE_NOT_ENABLED acceptable).
- STOP-IF-FAIL: discard the candidate AND tear down staging residue: `docker compose -p <slug>-mcp-staging --project-directory "$UPGRADE_DIR" -f docker-compose.yml -f docker-compose.staging.yml down --remove-orphans && ! docker ps -a --format '{{.Names}}' | grep -qx '<slug>-mcp-staging' || { echo 'FAIL: staging teardown left a container/route (live-credential exposure) — remove it before discarding candidate state'; exit 1; }` (removes the staging container + Traefik route + any orphans, and PROVES absence — mirrors the Phase 9.4 success teardown), `docker image rm "$(cat "$BASELINE_DIR/image-ref.txt")" 2>/dev/null || true`, then BRANCH on `$(cat "$BASELINE_DIR/mode")`: UPGRADE (`MODE=upgrade`) `git -C /opt/mcp-servers/<slug>-mcp worktree remove --force "$UPGRADE_DIR" && git -C /opt/mcp-servers/<slug>-mcp branch -D "$UPGRADE_BRANCH"` (PROD untouched, still old SIGNED -- NO production rollback). NET-NEW (`MODE=net-new` -- `$UPGRADE_DIR` IS the primary clone, there is NO separate worktree and nothing was ever promoted): TEAR DOWN to a clean slate -- `rm -rf /opt/mcp-servers/<slug>-mcp "$BASELINE_DIR/skill-staging"`; delete the GitHub repo only after AJ confirms. Root-cause, restart from STEP 7.1 (upgrade) or Phase 0-NEW (net-new).

**STEP 8.3** -- Sign the candidate (BOTH mirrors) on the upgrade branch (after verify; before the final gate so Check 3 is GREEN)
- Architecture precondition: `contract.yaml` is MOUNTED runtime config (NOT baked/`COPY`'d) so `pair_status` flips without a rebuild and staging `/health` re-reads it on restart. (Legacy baked-contract pairs: add the mount in Phase 3 STEP 3.2 first.)
- Action (set the canonical SINGLE status value on BOTH mirrors -- a precise whole-line SET, never a broad word-`sed` that could corrupt a multi-token status line): `sed -i 's/^pair_status:.*/pair_status: SIGNED/' "$UPGRADE_DIR/contract.yaml"; sed -i -E 's/^\*\*(Pair status|Status)\*\*:.*/**Pair status**: SIGNED/' "$UPGRADE_DIR/docs/pairing-contract.md" && [ "$(grep -cE '^\*\*Pair status\*\*: SIGNED$' "$UPGRADE_DIR/docs/pairing-contract.md")" = 1 ] || { echo 'FAIL: human contract not normalized to EXACTLY ONE canonical **Pair status**: SIGNED line (non-canonical header?) — refuse to sign with a stale human mirror'; exit 1; }; git -C "$UPGRADE_DIR" diff --quiet -- ':!contract.yaml' ':!docs/pairing-contract.md' ':!docs/trivy-scan-*' && git -C "$UPGRADE_DIR" diff --cached --quiet -- ':!docs/trivy-scan-*' || { echo 'FAIL: unexpected tracked changes since the Phase 7 build (checked BOTH unstaged AND staged) — at sign ONLY the two contract mirrors (unstaged seds) + the Phase 7.2 trivy evidence (staged) may differ; anything else means verified != shipped'; exit 1; } && git -C "$UPGRADE_DIR" commit contract.yaml docs/pairing-contract.md docs/trivy-scan-*.txt -m 'sign: <slug> v<NEW> SIGNED (post-staging-verify)' && git -C "$UPGRADE_DIR" show HEAD:contract.yaml | grep -q '^pair_status: SIGNED' || { echo 'FAIL: sign commit failed OR contract.yaml is not SIGNED at HEAD — refuse to record/promote (verified != shipped)'; exit 1; }; export SIGNED_COMMIT=$(git -C "$UPGRADE_DIR" rev-parse HEAD) && echo "$SIGNED_COMMIT" > "$BASELINE_DIR/signed-commit.txt" && IMAGE_TAG=v<NEW> docker compose -p <slug>-mcp-staging --project-directory "$UPGRADE_DIR" -f docker-compose.yml -f docker-compose.staging.yml restart <slug>-mcp` -- staging reloads the SIGNED mounted contract; image `.Id` unchanged (R11: both mirrors signed in lockstep so the human `.md` can never lag the machine `.yaml`).
- VERIFY: staging `/health.contract.pair_status` == SIGNED; `grep '^pair_status:' "$UPGRADE_DIR/contract.yaml"` is exactly `SIGNED`; the `.md` `**Pair status**:` line is exactly `SIGNED`; staging image `.Id` == `shipped-image-id.txt`.
- GATE: candidate SIGNED across both mirrors + git + staging `/health`; image `.Id` unchanged.

**STEP 8.4** -- FINAL 18-check Completion Gate ON STAGING (Check 3 signed GREEN)
- CONSULT: Bible `09_VERIFICATION_PROTOCOL.md § 1` (1-18) + `18_FRAMEWORK_REFINEMENTS_v19.1.11.md § Check Renumber` (R45->16, R49->17) + `19_FRAMEWORK_REFINEMENTS_v19.1.12.md § Check 18` (R50).
- Run every check 1-18 against the SIGNED STAGING artifact (Check 16 XOR Check 17 by architecture -- the non-applicable one is N/A by pattern; further N/A only with Tier 1 / R16 rationale). Per-pair MUST checks (probe STAGING):
  - **Check 3 -- Pairing Contract signed**: staging `/health.contract.pair_status` == SIGNED (signed in 8.3 -- this ordering is WHY signing precedes the final gate).
  - staging `/health` returns `{status, version: **v<NEW>**, framework: **<FRAMEWORK_NEW>**, contract.pair_status: SIGNED}` — assert version AND framework EXACTLY equal the expected new values (not merely present), else runtime drift passes staging and only surfaces post-cutover.
  - `tools/list` count matches `tool_inventory.target_count`.
  - **Two-scale cascade clean (R13, Check 10)** -- `(cd "$UPGRADE_DIR" && bash /root/aj-workspace/scripts/cascade-check.sh --static v<NEW> <FRAMEWORK_NEW> <slug>)` exits 0 (the candidate's file surfaces; the LIVE-`/health` form runs post-promotion in Phase 9.3 since the script's host is the prod subdomain).
  - All tools have `outputSchema` + `annotations` (Check 12.1+12.2); `MCP-Protocol-Version: 2025-11-25` negotiated.
  - every contract-declared banned tool (N per `banned_tools`) returns -32006 with migration message.
  - Identity probe (per Pairing Contract §10) returns expected account/org marker.
  - Static-arity audit clean (Check 16 two-layer / Check 17 all-in-one-class); Symmetric reader/writer audit clean (Check 15) where R48 applies.
  - **Per-tool timeout enforcement clean (Check 18, R50)** -- manual four-condition audit (Bible `19_*.md § Check 18`): install_annotator wraps via `asyncio.wait_for` with category-aware cap; every annotated `to_thread` flows through the wrap; contract.yaml `guardrail_timeout.enforcement_layers` declares tool-layer + session-layer; source pointers match. (`wait_for_audit.py` automation deferred to v19.1.13 -- run the audit manually until then.)
- GATE: 18/18 GREEN on staging (Check 16 XOR 17 architecture-N/A; further N/A only where R16 permits). The candidate is now VERIFIED + SIGNED + gated.
- STOP-IF-FAIL: discard the candidate + tear down staging residue (same procedure as STEP 8.2 STOP-IF-FAIL: `down` the staging project, remove the candidate image, remove the worktree + branch); PROD untouched. Root-cause, restart from STEP 7.1.

**STEP 8.5** -- (Tier 2+ only) Paired-vs-unpaired evaluation
- CONSULT: Bible `11_EVALUATION_HARNESS.md` (battery + paired/unpaired comparative report).
- Action: run `eval/battery.yaml` paired (skill autoload on) and unpaired; compare.
- GATE: paired run meets battery thresholds (Tier 1 may waive with rationale).

---

### PHASE 9 — Production Cutover (PROMOTE the verified + signed candidate; prod goes old-SIGNED -> new-SIGNED directly)

> The candidate is already VERIFIED + SIGNED + 18/18-gated on staging (Phase 8). Phase 9 ONLY promotes it. PROD is never exposed to a DRAFT or unverified candidate -- it transitions old-SIGNED -> new-SIGNED in one step.

**STEP 9.1** -- Pre-cutover .env hash check (final)
- Action: `NEW=$(sha256sum /opt/mcp-servers/<slug>-mcp/.env | awk '{print $1}'); BASE=$(cat $BASELINE_DIR/env-hash.txt | awk '{print $1}'); [[ "$NEW" == "$BASE" ]] || exit 1`
- GATE: `.env` unchanged from Phase 0.2 baseline.

**STEP 9.2** -- Promote: merge the SIGNED candidate into live + deploy the same image `.Id`
- Action 1 -- PRE-STAGE assert (refuse to BEGIN a cutover we cannot finish OR roll back -- runs BEFORE the Action 2 stop): `MODE=$(cat "$BASELINE_DIR/mode" 2>/dev/null); test -f "$BASELINE_DIR/signed-commit.txt" && test -d "$BASELINE_DIR/skill-staging" && [ "$(docker image inspect "$(cat "$BASELINE_DIR/image-ref.txt")" --format '{{.Id}}' 2>/dev/null)" = "$(cat "$BASELINE_DIR/shipped-image-id.txt")" ] && { if [ "$MODE" = net-new ]; then test ! -e /root/.claude/skills/<slug> && test ! -e /home/claude/.claude/skills/<slug>; else test -f "$BASELINE_DIR/pre-upgrade-commit.txt" && test -d "$BASELINE_DIR/skill-pre-upgrade" && docker image inspect <slug>-mcp-<slug>-mcp:v<CURRENT>-pre-upgrade >/dev/null 2>&1 && test -z "$(git -C /opt/mcp-servers/<slug>-mcp status --porcelain=v1 --untracked-files=all)" && [ "$(git -C /opt/mcp-servers/<slug>-mcp rev-parse HEAD)" = "$(cat "$BASELINE_DIR/pre-upgrade-commit.txt")" ] && git -C /opt/mcp-servers/<slug>-mcp merge-base --is-ancestor "$(cat "$BASELINE_DIR/pre-upgrade-commit.txt")" "$(cat "$BASELINE_DIR/signed-commit.txt")" && diff -rq /root/.claude/skills/<slug> "$BASELINE_DIR/skill-pre-upgrade" >/dev/null 2>&1; fi; } || { echo "FAIL: cutover prerequisites incomplete (mode=$MODE) -- signed commit + staged skill + the v<NEW> image STILL resolving to its staging-verified .Id (no retag/rebuild during the soak — checked BEFORE the stop so a mismatch never leaves prod down mid-cutover) are required for EVERY pair; an UPGRADE additionally requires the FULL rollback bundle (pre-upgrade-commit.txt + skill-pre-upgrade + the v<CURRENT>-pre-upgrade image) AND a CLEAN live worktree still at the EXPECTED pre-upgrade HEAD with the signed commit FAST-FORWARDABLE AND the live skill still BYTE-MATCHING the Phase 5 backup (no source OR skill drift during the soak — else the post-stop ff-only merge would fail mid-outage, or the staged-skill rsync would SILENTLY ERASE operator edits made during the soak). a NET-NEW additionally requires BOTH live skill paths (/root + /home) to still be ABSENT (fresh first placement — a collision created since Phase 0N.1 would be rsync --delete-clobbered with no rollback baseline). Refuse to stop prod for a cutover that cannot be COMPLETED or rolled back."; exit 1; }` (the Layer-2 autoloader registry is OPTIONAL: only pairs that declared Layer 2 staged a `$BASELINE_DIR/skill-autoloader-registry.json.staged`, applied conditionally in Action 4 — its absence is normal, not a failure)
- Action 2 -- BLOCK invocation (fail-CLOSED maintenance gate, so the MCP + skill + hooks can swap as ONE transaction and the agent can NEVER hit a half-swapped pair -- new-MCP+old-skill or the reverse -- because a dead endpoint is an honest error while a tool-signature mismatch is silent corruption): `docker compose -p <slug>-mcp --project-directory /opt/mcp-servers/<slug>-mcp stop <slug>-mcp`. Nothing serves until every surface is consistent.
- Action 3 -- MCP source + contract -> SIGNED (first+only mutation of the live mount; lands ALREADY SIGNED so PROD never reports DRAFT): `git -C /opt/mcp-servers/<slug>-mcp merge --ff-only "$(cat "$BASELINE_DIR/signed-commit.txt")"` (the Phase 8.3 SIGNED commit, persisted so it survives a multi-day soak).
- Action 4 -- apply the PAIRED skill + hooks WHILE the endpoint is still gated, so when the MCP comes up next it is ALREADY matched to the new skill: `rsync -a --delete "$BASELINE_DIR/skill-staging/" /root/.claude/skills/<slug>/` then R11 mirror `rsync -a --delete /root/.claude/skills/<slug>/ /home/claude/.claude/skills/<slug>/` (no-op where hardlinked) then, ONLY IF Layer 2 was declared, install the staged registry: `{ [ ! -f "$BASELINE_DIR/skill-autoloader-registry.json.staged" ] || cp -a "$BASELINE_DIR/skill-autoloader-registry.json.staged" /root/.claude/hooks/skill-autoloader-registry.json; }` (returns 0 — a TRUE no-op — when Layer 2 is absent, so a gated/`set -e` cutover never aborts on the normal absent-file case).
- Action 5 -- deploy + UNBLOCK: bring up the SAME staging-verified image `.Id` (`-p <slug>-mcp` pins the prod project so the name matches Phase 7) -- the endpoint resumes serving ONLY now, with new MCP + new skill already in lockstep — and the staging-verified `.Id` was already re-confirmed in Action 1 BEFORE the stop (so a retag/rebuild during the soak is caught while prod is still UP, never mid-outage): `IMAGE_TAG=v<NEW> docker compose -p <slug>-mcp --project-directory /opt/mcp-servers/<slug>-mcp up -d <slug>-mcp`. On ANY failure in Actions 3-5 the pair stays gated -> jump to the STEP 9.3 STOP-IF-FAIL all-surface rollback (image + both contract mirrors + skill + hooks + registry), which also restarts the endpoint at the prior SIGNED state.
- VERIFY: deployed container image `.Id` == `$BASELINE_DIR/shipped-image-id.txt` (verified == shipped) AND `docker inspect <slug>-mcp --format='{{.State.Health.Status}}'` healthy within 60s AND the live skill `version`/`mcp_tool_count` now match the candidate.
- GATE: container healthy + image `.Id` matches the staging-verified artifact.
- STOP-IF-FAIL: image-ID mismatch = someone rebuilt -> STOP (unverified artifact). Otherwise full rollback (see 9.3).

**STEP 9.3** -- Post-cutover PRODUCTION confirmation (live)
- VERIFY (against PROD live): `/health` == `{status, version: v<NEW>, framework, contract.pair_status: SIGNED}`; full live cascade gate (run FROM the prod compose dir so it reads prod `docker-compose.yml`/`contract.yaml`, never a candidate worktree's files) `(cd /opt/mcp-servers/<slug>-mcp && bash /root/aj-workspace/scripts/cascade-check.sh v<NEW> <FRAMEWORK_NEW> <slug>)` exits 0 (fail-closed, hits prod `/health`); a FINAL workhorse smoke 100% on prod (Bible §09:210 -- confirms the cutover artifact serves live upstream); live image `.Id` == `shipped-image-id.txt`.
- Action (sync the SIGNED live `main` to origin so GitHub == prod — Law 12 reproducibility; fail-closed, runs BEFORE the STEP 9.4 worktree/branch teardown): `git -C /opt/mcp-servers/<slug>-mcp push origin main && [ "$(git -C /opt/mcp-servers/<slug>-mcp ls-remote origin -h refs/heads/main | awk '{print $1}')" = "$(git -C /opt/mcp-servers/<slug>-mcp rev-parse HEAD)" ] || { echo 'FAIL: origin/main did not advance to the signed prod commit — GitHub is STALE vs prod (Law 12); resolve before teardown/closure'; exit 1; }`
- GATE: prod live = SIGNED + version v<NEW> + cascade clean + workhorse 100% + origin/main == prod HEAD = **production sign-off COMPLETE**.
- STOP-IF-FAIL: rollback BRANCHES on the DURABLE `mode` marker -- `MODE=$(cat "$BASELINE_DIR/mode")` -- so UPGRADE-vs-NET-NEW is EXPLICIT, never inferred from a file's survival (a /tmp cleanup could otherwise misclassify an upgrade and `rm -rf` a live pair). A missing or unknown `mode` => STOP for MANUAL rollback, NEVER auto-`rm -rf`. **UPGRADE** (`MODE=upgrade`) -- revert in the SAME GATED ORDER as the forward cutover, so there is NEVER an old-MCP/new-skill window: (1) GATE `docker compose -p <slug>-mcp --project-directory /opt/mcp-servers/<slug>-mcp stop <slug>-mcp`; (2) reset source+contract `git -C /opt/mcp-servers/<slug>-mcp reset --hard "$(cat "$BASELINE_DIR/pre-upgrade-commit.txt")"` (back to pre-upgrade SIGNED); (3) restore the PAIRED skill + hooks WHILE STILL GATED — `rsync -a --delete "$BASELINE_DIR/skill-pre-upgrade/" /root/.claude/skills/<slug>/` + R11 mirror + autoloader registry ONLY if Layer 2: `{ [ ! -f "$BASELINE_DIR/skill-autoloader-registry.json.pre" ] || cp -a "$BASELINE_DIR/skill-autoloader-registry.json.pre" /root/.claude/hooks/skill-autoloader-registry.json; }` (true no-op when absent); (4) ONLY NOW redeploy the prior image `IMAGE_TAG=v<CURRENT>-pre-upgrade docker compose -p <slug>-mcp --project-directory /opt/mcp-servers/<slug>-mcp up -d <slug>-mcp` — it comes up ALREADY matched to the restored prior skill. Image + config + skill + hooks now match the prior-cycle SIGNED state; root-cause, restart from STEP 7.1. **NET-NEW** (`MODE=net-new` ONLY — there is NO prior SIGNED state to revert to, so a failed first cutover rolls back by TEARING DOWN to a clean slate, never leaving a half-live first pair): `docker compose -p <slug>-mcp --project-directory /opt/mcp-servers/<slug>-mcp down --rmi local -v && ! docker ps -a --format '{{.Names}}' | grep -qx '<slug>-mcp' && rm -rf /opt/mcp-servers/<slug>-mcp /root/.claude/skills/<slug> /home/claude/.claude/skills/<slug> || { echo 'FAIL: net-new teardown aborted — `down` failed OR a routed container survives; refuse to rm the repo while a container may still serve (orphan/route-leak risk) — resolve manually'; exit 1; }`; remove any staged autoloader entry from the live registry; remove the pair's Traefik route/subdomain; delete the GitHub repo only after AJ confirms. Either branch ends at a COHERENT state (prior SIGNED, or nothing) — root-cause, restart from STEP 7.1 (upgrade) or Phase 0-NEW (net-new).
- Action: record the completed sign-off in vault + ST (Phase 10.3 sinks) with the gated image `.Id` as evidence

**STEP 9.4** -- Tear down STAGING + the upgrade worktree (SUCCESS path — pristine environment + security-by-default)
- Precondition: STEP 9.3 GATE passed (prod live = SIGNED v<NEW> + workhorse 100%). ONLY after prod is confirmed do we destroy the staging copy — until then it is the rollback-comparison reference.
- Action (stop + remove the PUBLIC staging deployment so `<slug>-mcp-staging.arjtech.in` no longer runs with LIVE upstream credentials after sign-off): `IMAGE_TAG=v<NEW> docker compose -p <slug>-mcp-staging --project-directory "$UPGRADE_DIR" -f docker-compose.yml -f docker-compose.staging.yml down --remove-orphans`
- VERIFY (prove ABSENCE, not mere unreachability): PRIMARY (authoritative) — the staging container is fully REMOVED, which (Traefik Docker-provider derives routes from running containers) deterministically drops its router/service: `! docker ps -a --format '{{.Names}}' | grep -qx '<slug>-mcp-staging'`. SECONDARY (confirmation only — a DNS/TLS/network error could ALSO make this pass, so it is NOT sufficient alone): `if curl -fsS -o /dev/null https://<slug>-mcp-staging.arjtech.in/health; then echo 'FAIL: staging route still serves after teardown'; exit 1; fi` (curl SUCCESS = route still up = fail; an if-guard is `set -e`-safe where `curl; test $?` would abort on the expected curl failure).
- Action (remove the isolated upgrade worktree — its job is done; the SIGNED commit is already merged into the live mount): for UPGRADE (`MODE=upgrade`) `git -C /opt/mcp-servers/<slug>-mcp worktree remove "$UPGRADE_DIR" && git -C /opt/mcp-servers/<slug>-mcp worktree prune && { git -C /opt/mcp-servers/<slug>-mcp branch -D "$UPGRADE_BRANCH" 2>/dev/null || true; } && { git -C /opt/mcp-servers/<slug>-mcp push origin --delete "$UPGRADE_BRANCH" 2>/dev/null || true; }` (the remote branch delete is gated behind successful local worktree removal+prune — a failed `worktree remove` stops the whole chain, never falling through to delete the branch (also prune the now-merged upgrade branch local+origin — feedback_prune_merged_branches). SKIP for `MODE=net-new` (`$UPGRADE_DIR` IS the live repo, not a separate worktree).
- GATE: zero staging container/router/host remains + upgrade worktree removed (upgrades) = environment pristine. STOP-IF-FAIL: a lingering staging surface with live credentials is a SECURITY exposure — remove it before declaring closure.

---

### PHASE 10 — Closure (within 48h post-cutover)

**STEP 10.1** — Pristine Sweep (7 categories)
- CONSULT: Bible `09_VERIFICATION_PROTOCOL.md § 3` (full 7-category sweep spec)
- RETENTION-GATED prune of the DURABLE run-state dir `/var/lib/mcp-upgrade-state/<slug>-<DATE>` (it holds the rollback bundle): DELETE it ONLY after BOTH (a) STEP 9.3 sign-off is recorded (10.3 vault writeback) AND (b) a 24h post-cutover stability window has elapsed with NO rollback. NEVER prune while a pair is mid-soak or inside the 24h window — premature deletion destroys rollback evidence. Once BOTH conditions hold: `rm -rf /var/lib/mcp-upgrade-state/<slug>-<DATE>`
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
2. **Never guess at upstream API specs** (R38) — verify or cite docs URL.
3. **Never skip Phase 0.5 Census** — it's how you find pre-existing baseline bugs that wrapping doesn't fix.
4. **Never ship a tool without R38 per-tool upstream-verification** — `verified_at` timestamp OR vendor docs URL in source.
5. **Never modify .env** — Pre-flight Rule 0 absolute.
6. **Never SIGN without staging workhorse smoke 100% + R39 census = 0 TRUE_BUG** (Phase 8.2, Bible §09:208); **never PROMOTE to prod without the 18/18 final gate GREEN on the signed staging artifact** (Phase 8.4, incl. Check 3 "Pairing Contract signed"; Check 16 XOR 17 architecture-N/A). Sign at 8.3 — between staging verify (8.2) and the staging final gate (8.4); prod is never exposed as DRAFT.
7. **Never duplicate Bible content into this skill** — replace with smart reference.

## Validation (quick-check)

```bash
# Ten Laws quick check (per Bible §01)
# Laws: 1 __new__ present; 3 no run_in_executor; 5 no **kwargs in tools/; 6 stateless_http=True;
# 7 no OAuthProvider (Pattern-2 ban); fail-fast on_duplicate="error". (comments above so `\` stays the last char)
grep -rn "__new__" src/ >/dev/null \
  && ! grep -rqn "run_in_executor" src/ \
  && ! { grep -rlnE "\*\*kwargs" --include='*.py' src/ 2>/dev/null | grep -qE '(^|/)tools/'; } \
  && grep -rn "stateless_http=True" src/ >/dev/null \
  && ! grep -rqn "OAuthProvider" src/ \
  && grep -rn 'on_duplicate="error"' src/ >/dev/null \
  && echo "Ten Laws: PASS"

# Pairing Contract present (Bible §07 — Law 8)
test -f <pair-repo>/docs/pairing-contract.md && echo "Contract: in pair repo"
ls tests/guardrails/test_g*.py | wc -l | grep -qE "^(10|11)$" && echo "Guardrails: tests present"   # Law 9

# Check 12 — MCP 2025-11-25 spec-compliance (Bible §09 Check 12)
curl -fsS -X POST https://<pair>-mcp.arjtech.in/mcp -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' -H 'MCP-Protocol-Version: 2025-11-25' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}' | sed -n 's/^data: //p' | \
  jq '.result.tools | map(.outputSchema != null) | all'                # 12.1 — all have outputSchema
# 12.4 — primitive registration: assert each APPLICABLE primitive calls .register(mcp).
# Elicitation is conditional (MUST only if the pair has interactive bootstrap flows), Sampling is MAY
# (Bible §03 §6) — do NOT fail a non-interactive pair for a missing elicitation register.
# SMOKE HELPER ONLY (warns, never fails). The HARD applicable-primitive gate is Phase 4.6 / final-gate Check 12.4 — it fails on any instantiated-but-unregistered OR missing-applicable primitive.
for prim in resources prompts; do grep -qE "${prim}.*\.register\(mcp\)" src/<slug>_mcp/server.py || echo "WARN(smoke): $prim not registered — confirm applicability at Phase 4.6"; done
# elicitation: conditional — pass if the pair has NO interactive flow, else require registration
! grep -qE 'elicitation' src/<slug>_mcp/server.py || grep -qE 'elicitation.*\.register\(mcp\)' src/<slug>_mcp/server.py \
  || { echo "FAIL: elicitation present but not registered"; exit 1; }

# Two-scale cascade (R13, Bible §09 Check 10) — server vs framework are INDEPENDENT; pass each its own value.
# Run from the pair compose dir /opt/mcp-servers/<slug>-mcp. Exit 0 = aligned, exit 1 = drift (fail closed).
EXPECTED_SERVER=v<EXPECTED>; EXPECTED_FRAMEWORK=<FRAMEWORK_NEW>; PAIR=<slug>   # PLACEHOLDERS — fill from contract.yaml (mcp_image_tag / framework_version) or /health; NOT copyable literals
# cwd-explicit: the script reads compose/contract from CWD + curls the prod subdomain, so run it FROM the live pair dir (a subshell keeps cwd local). Use --static from the candidate worktree pre-deploy.
( cd "/opt/mcp-servers/${PAIR}-mcp" && bash /root/aj-workspace/scripts/cascade-check.sh $EXPECTED_SERVER $EXPECTED_FRAMEWORK $PAIR ) \
  || { echo "FAIL: cascade drift across the two scales"; exit 1; }

# Static-arity audit — run EXACTLY ONE per the pair's declared architecture (an ARCH switch, so a literal copy can never run both nor the wrong one): R45 for two-layer, R49 for all-in-one-class.
ARCH=$(grep -oE 'pattern:[[:space:]]*(two-layer|all-in-one-class|self-contained)' contract.yaml 2>/dev/null | grep -oE '(two-layer|all-in-one-class|self-contained)' | head -1); [ -n "$ARCH" ] || { echo 'set ARCH=two-layer|all-in-one-class|self-contained (not derivable from contract.yaml architecture.pattern) — never default-guess the audit'; exit 1; }   # DERIVED from contract.yaml architecture.pattern (Phase 2.4 declares architecture as an object), fail-closed if absent
if [ "$ARCH" = self-contained ]; then
  echo "self-contained pattern — NO static-arity audit applies (R39 census + Codex hard-gate cover arity); skipping R45/R49"
elif [ "$ARCH" = two-layer ]; then
  # R45 wrapper audit (script EXISTS in the Bible) — two-layer pairs ONLY
  python3 /root/aj-workspace/MCP-God-Agent-Development-Bible/framework/scripts/wrapper_arity_audit.py --server-file ... --client-file ... \
    || { echo "FAIL: R45 wrapper arity drift"; exit 1; }
else
  # R49 intra-class audit — all-in-one-class pairs ONLY. intra_class_arity_audit.py authoring is PENDING in the Bible (AST pseudocode shipped).
  # Split missing-script (-> recorded manual audit) from present-but-FAILING (-> hard fail; never mask arity drift):
  if test -f /root/aj-workspace/MCP-God-Agent-Development-Bible/framework/scripts/intra_class_arity_audit.py; then
    python3 /root/aj-workspace/MCP-God-Agent-Development-Bible/framework/scripts/intra_class_arity_audit.py --class-file ... --class-name ... \
      || { echo "FAIL: R49 intra-class arity drift"; exit 1; }
  else
    { test -s "$BASELINE_DIR/r49-manual-audit.txt" && grep -q "$UPGRADE_COMMIT" "$BASELINE_DIR/r49-manual-audit.txt" && grep -qE '^(PASS|RESULT: PASS)' "$BASELINE_DIR/r49-manual-audit.txt"; } || { echo "FAIL: R49 manual audit at \$BASELINE_DIR/r49-manual-audit.txt is missing, EMPTY, not tied to candidate commit \$UPGRADE_COMMIT, or lacks an explicit PASS line — run the intra-class arity audit per Bible §18 pseudocode and record the candidate commit + PASS/findings (mere file existence is NOT a pass)"; exit 1; }
    echo "R49 script pending — using recorded manual audit: $BASELINE_DIR/r49-manual-audit.txt"
  fi
fi

# Post-deploy drift (event-driven, NOT calendar cron — R14)
DRY_RUN=1 /root/aj-workspace/scripts/drift-check.sh --pair=<slug>
```

## Common Mistakes — quick triage (full list lives in Bible chapters)

For the full anti-pattern catalog, consult: Bible `00_PHILOSOPHY.md` (principles), `01_INVIOLABLE_LAWS.md` (laws + Common Mistakes table), `14_*.md § R38` (upstream-verification banned patterns). High-frequency triage:

| Symptom | Where to look in Bible |
|---|---|
| `TypeError: takes N positional arguments but M were given` | `16_*.md` (R45 two-layer) or `18_*.md` (R49 all-in-one-class) |
| `ValueError: structured_content must be a dict or None` | `14_*.md § R40` Result-Shape Wrapping |
| `ImportError: FastMCP background tasks require the 'tasks' extra` | `06_STACK_MANIFEST.md` (use `fastmcp[tasks]>=3.0.2`) |
| Cascade-check fails on framework_version mismatch | `14_*.md § R13` (atomic surface bump) |
| Blanket 401 after auth change | `01_INVIOLABLE_LAWS.md § Law 7` (Pattern 2 + OAuthProvider ban) |
| Tool returns success on REST 4xx/5xx | semantic gate caught Codex-class find — wrapper boundary status propagation. No AST audit detects this; rely on R39 + Codex review. |
| Operator-intent tool needs N round-trips | `17_FRAMEWORK_REFINEMENTS_v19.1.10.md § R48` Complete-Capability Invariant |
| Symmetric metadata reader missing for writer | `17_*.md § Check 15` Symmetric Reader Audit |
| Container `.bible` label lags Bible main | this skill's Bible-pull discipline + Phase 1 Bible-sync minor patch |

## Workflow Integration

- **Pristine Sweep**: 7 categories per Bible `09_VERIFICATION_PROTOCOL.md § 3`, post-upgrade.
- **Completion Gate**: 18 checks per Bible `09 § 1` (v19.1.12; Check 16 XOR 17 by architecture, Check 18 = R50). Order = Phase 8.2 staging verify → 8.3 sign → 8.4 final 18/18 gate (incl. Check 3 signed), ALL on staging; Phase 9 promotes the verified+signed candidate. See Session Rule 7.
- **Decision journal**: every upgrade records `mcp__sequential-thinking__record_decision` with `tags: mcp-upgrade,<pair>,<outcome>,st-derived` per Tier-1 Directive 5.
- **Vault writeback**: completion → `vault_submit_knowledge` with pair slug + framework version.
- **Session-learnings**: post-upgrade append `[UPGRADE] <pair> Bible-v<X> server-v<Y>` entry with non-obvious learnings.

## Session Operating Rules — zero lapses

1. **Pull canonical Bible first** (this skill's prologue protocol). Container labels and skill copies are not sources of truth.
2. **Skill = orchestration, Bible = content.** Every CONSULT line is a smart reference to Bible. Never restate Bible content in this skill.
3. **Pair-repo discipline** — all pair-specific artifacts in pair repo (`docs/pairing-contract.md`, `docs/pilot-learnings.md`, `docs/upstream-sync-<date>.md`). Never pollute the Bible.
4. **Two-scale version clarity** — always state which scale you're bumping (server vs framework). Never conflate.
5. **Zero-deferred-must-adopt** — sync recommendations get implemented this upgrade or waived with expiry.
6. **Cascade-check (two forms, R13 / Bible §09 Check 10)** — PRE-deploy: `cascade-check.sh --static v<NEW> <FRAMEWORK_NEW> <slug>` from the candidate worktree (file surfaces only — live is still OLD). POST-promotion (Phase 9.3): full `cascade-check.sh v<NEW> <FRAMEWORK_NEW> <slug>` against the now-live pair (incl /health). `<FRAMEWORK_NEW>` already carries `Bible-v`. Exit 0 = aligned, exit 1 = drift. NEVER run the live form pre-deploy.
7. **Verify → sign → final-gate, ALL on staging; then promote** — on STAGING: Phase 8.2 verify (workhorse 100% + census 0 TRUE_BUG, §09:208) → Phase 8.3 sign (canonical whole-line set on BOTH mirrors + restart, same image `.Id`) → Phase 8.4 final 18/18 gate (Check 3 GREEN). Phase 9 then PROMOTES the verified+signed candidate to prod (old-SIGNED → new-SIGNED, never DRAFT). Signing is a config reload, never a rebuild; the Phase-7 image `.Id` is what verifies and ships.
8. **Tier-appropriate, not tier-maximum** — R16 banned gold-plating (Tier 3 requirements on Tier 1 pair).
9. **Event-driven drift only** — no nightly cron for stable production pairs (R14).
10. **Post-cutover Phase 10** — quality audit within 48h. Findings → `<pair-repo>/docs/pilot-learnings.md` + framework refinements upstream-cascaded to Bible if they generalize.
11. **Never claim done without running the Gate** — cheerleading is a drift class. Every `PASS` needs a command output.
12. **Skill ↔ Bible sync** — when Bible bumps, this skill's version markers + Refinements Map + Bible chapter map update within the same session. Drift = paired-unit failure.
