# 06 — Consolidated Ultra-Plan Handoff Brief

**Date:** 2026-04-30
**Branch:** `enhance/better-adr-capture` (rebase onto `main` is move M2)
**Destination:** Claude ultra-plan mode via `gsd-ultraplan-phase`
**Status:** consolidated, NOT summarized. Detail and nuance preserved per user standing rule (CLAUDE.md: "summarization is HARMFUL when producing technical normative outputs").
**Supersedes:** `04-integration-plan.md` (rejected by user — "the old plan is crap"). Wave numbering from prior draft is **dropped**; moves are presented as a dependency graph.

---

## Reading Order for Ultra-Plan

1. §1 Source research — what's authoritative (user's research is centered).
2. §6 Constraints/invariants — bind every move.
3. §7 Anti-patterns — what Claude got wrong; do NOT repeat in ultra-plan.
4. §2 Right data flow — the substrate model.
5. §3 Right components — what to build/promote/deprecate.
6. §4 Right moves with dependency graph — the action set.
7. §5 Substrate defects — the deficiency catalogue moves must close.
8. §8 Open questions — remaining, do NOT close in plan.

---

## §1 Source Research (the "best parts of MY research", preserved)

### 1.1 User's research artifacts (centered, authoritative)

**Architecture repo — `/home/randyg/repos/architecture/`:**
- `elt/` — first-attempt monolithic adocs needing extraction:
  - `elt/extract-section.sh` (bash) — mines a single `[#{idprefix}<id>]` section. Workaround for adocs too large to process naively (`conops.adoc` = 96 KB, `data-models.adoc` = 41 KB).
  - `elt/*.adoc` — 5 large adocs: conops, data-models, subsystems, use-cases, user-stories. `.adoc-index` files alongside.
  - `elt/extracted/*.yaml` — 4 extracted YAMLs: data-models (355 lines), subsystems (38), use-cases (1669), user-stories (152).
- `fw/` — refactored one-file-per-artifact, organized in subsystem subdirs (`fusion/`, `bin/`, `lens/`, `pipeline/`, `workspace/`, `knarrative/`, `harvester/`, `knote/`, `manual/`, `smartmap/`):
  - ID pattern: `{TYPE}-FWA-{NN}-{slug}.adoc` where TYPE ∈ {UC, US, SS}.
  - `fw/extracted/{fwa-subsystems, fwa-user-stories, fwa-use-cases}.yaml` — same extraction, smaller per-file because source adocs are smaller.
- `tools/parsers/parse-{uc,us,ss,dm}.awk` — 4 awk scripts, 524 lines total, adoc-type-aware extractors (UC, US, SS, DM).

**Pattern (likely an evolution):** elt = first-attempt → fw = refactored. Direction: **structured AsciiDoc → YAML, scripts as glue, schema emerging from project usage.**

**YAML schema observed in `fw/extracted/fwa-user-stories.yaml` (REFERENCE — current state, has gaps):**
```yaml
- id: "US-FWA-12"
  title: "..."
  acceptance_criteria:
    - version: "current"
      text: "..."
```
- Versioned acceptance criteria (`version: "current"`) — superior to vanilla markdown PRD prose.
- **Lost-rationale failure surface:** schema has `id`, `title`, `acceptance_criteria` only. Missing `rationale`, `source`, `traces_to`, `phase`, `strength`, `test_procedure` — all of which were in user's PROPOSED schema (Factweave 2026-03-24, see §1.4).

**User's proposed schema (Factweave 2026-03-24, partially implemented):**
```yaml
FR-FWA-KA-001:
  system: FWA
  subsystem: KA
  requirement: "..."
  source: PRD1+FRI
  phase: MVP
  strength: Strong
  rationale: "..."         # ← rationale was IN the proposed schema
  traces_to:
    - journey: Elena-J1
    - principle: P2-facts-are-claims
  test_procedure: TP-FWA-KA-001
```
- ID convention: `{TYPE}-{SYSTEM}-{NN}` — TYPE ∈ {SS, UC, US, DM, FR, AC}, SYSTEM ∈ {FWA, ELT, …}.
- This is the "closest to ideal but semantically weaker than it should be" baseline per A12.

**Linkml project — `/home/randyg/repos/linkml/`:**
- `_bmad-output/learnings/draft-bmad-improvement-analysis.md` — **1434 lines, 27 sections**. Detailed deficiency analysis, contribution-readiness assessment, prioritized next actions.
- `_bmad-output/learnings/bmad-upstream-contribution-strategy.md` — **632 lines, 12 sections**. Categorization framework, priority improvements with G/I/C scoring, execution plan, risk assessment.
- Promoted Jinja2/Mustache adoc templates to (now-deprecated) `lkml-gen/src/lkml_gen/generators/templates/`. **All `lkml-gen` references are deprecated; current tool is `lkml`.**

**G/I/C scoring rubric** (Generality / Impact / Cost-of-skipping, each 0-5, max 15) — adopt this; do NOT re-derive a parallel taxonomy:

| Cluster | Score | Maps to |
|---|---|---|
| Cluster 0 — Workflow Resumption Metadata (frontmatter + resumption section + session naming) ⭐ | 15/15 | E2 |
| Cluster 3 — BMAD Governance VCS Strategy ⭐ | 15/15 | E1 / E4 |
| Cluster 4 — Party Mode Memory Loss Anti-Pattern ⭐ | 15/15 | E1 + E4 generalized |
| Cluster 1 — PRD Context Budget Management | 14/15 | token bloat |
| Cluster 2 — Architecture Prerequisite Enforcement | 13/15 | E2 prerequisite |
| Cluster 6 — Agent Customization Patterns | 12/15 | FSM scope |
| Cluster 5 — Progressive Help System | TBD | help skill |
| Cluster 7 — Domain-Specific Workflow Steps | TBD | domain skills |

**Contribution-readiness % (linkml's confidence for upstreaming):**

| Improvement | Confidence | Note |
|---|---|---|
| Workflow Resumption Metadata | 100% | CRITICAL, enables safe compaction/clear |
| In-Session Decision Persistence | 95% | Pure methodology |
| Workflow Step Output Validation | 90% | Generic completion gate |
| Context Budget Management | 90% | Generic synthesis pattern |
| Prerequisite Workflow Enforcement | 85% | Generic sequencing |
| Completion Protocol | 70% | Needs project-agnostic config format |
| GitOps Protocol | 75% | Git-specific, not universal VCS |
| Tooling Requirements | 65% | Many project-specific details |
| Worktree Awareness | 60% | Highly project-specific (monorepo) |

**Cross-project artifacts (the user's coordination primitive):**
- `liaison/` mailbox protocol — appears in BOTH `linkml/` and `archbox/`. Cross-project coordination primitive. Not in any taxonomy yet. Candidate new BMAD skill `bmad-liaison` if promoted.
- `failings/` and `learnings/` peripheral output dirs — appear in linkml; semantic siblings of `liaison/`.

**ENCOSE-aligned requirements engineering practice (per user, Factweave 2026-03-24):**
- Subsystem-prefixed IDs — `FR-FWA-001`, not bare `FR-001`. Stable across documents.
- Requirement traceability cradle to grave through test procedures back to provenance.
- Capability-area organization as primary unit, not flat enumeration.
- Versioned acceptance criteria (`version: "current"` vs `"future"`).
- No lossy transformation — adoc carries semantic relationships (cross-references, "Includes VM-US-001, VM-US-040, VM-US-043, …") that flat YAML must preserve.
- BMAD's default (`FR-001` restart-per-document) is "ROOKIE move in serious system engineering."

**Review-edit-extract-diff loop (the user's collaboration pattern, per 2026-04-29 pointer):**
- Operator edits authoritative source (adoc/md/YAML).
- Agent extracts/normalizes YAML.
- System diffs new YAML against prior committed YAML.
- Operator reviews diff as the change-proposal artifact (= PR diff for code review).
- Operator accepts/rejects/refines.
- Generalizes to ANY agent/operator collaboration cycle. Substrate-invariant.

### 1.2 BMAD substrate evidence

**Repo restructured** since the evolution docs were written: `src/modules/bmm/` no longer exists. Current shape:
- `src/bmm-skills/{1-analysis, 2-plan-workflows, 3-solutioning, 4-implementation}/` — phase directories of native skill packages.
- `src/core-skills/` — bmad-* shared skills (advanced-elicitation, brainstorming, party-mode, etc.).

Each skill package: `SKILL.md`, `bmad-skill-manifest.yaml`, `workflow.md`, optional `steps/`, optional `templates/`, `data/`.

**All four evolution fix files in `fixes/evolution-{1..4}/` reference the OLD `_bmad/bmm/workflows/...` paths.** Must be retargeted onto `src/bmm-skills/`.

**Evolution audit verdicts (full report: `04-evolution-audit.md`, 436 lines):**
- **E1 — Workflow Execution Discipline (FACILITATOR / A/P/C / stepsCompleted / FORBIDDEN-load-next / NEVER-generate-without-input):**
  - APPLIED (full 5-marker pattern): `bmad-create-architecture`, `bmad-create-ux-design`, `bmad-create-prd` (steps-c), `bmad-create-product-brief`. Research workflows use deliberate `[C]-only` variant (4 markers, no A/P/C — by design).
  - **MISSING:** `bmad-correct-course/` (zero FACILITATOR/A/P/C markers), `bmad-check-implementation-readiness/` (zero markers across all 6 step files). **Highest-leverage upstreaming target.** These are precisely the skills the original Evolution-1 fix files (`fixes/evolution-1/checklist.md`, `step-02-prd-analysis.md`, `step-06-final-assessment.md`) were authored for.
- **E2 — Workflow Resumption Metadata (state-recovery section + session naming + step-01b-continue):**
  - 4 step-01b-continue.md files exist with correct STATE-RECOVERY semantics: `bmad-create-product-brief`, `bmad-create-prd`, `bmad-create-ux-design`, `bmad-create-architecture` — but reached only via implicit step-01-init routing.
  - **NO workflow.md anywhere contains the explicit `State Recovery Check` section** called for in spec. Gate not authoritative even where step-01b exists.
  - All 5 original P0/P1/P2 targets carry NO step-01b file: `bmad-create-epics-and-stories`, `bmad-check-implementation-readiness`, `bmad-generate-project-context`, `bmad-sprint-planning`, `bmad-create-story`.
  - **Highest-leverage:** `bmad-create-epics-and-stories` (workflow.md gate + step-01b-continue.md + step-01 Section-0 guard).
- **E3 — Workflow Status Registry (`bmm-workflow-status.yaml`):**
  - SPEC UNIMPLEMENTED in `src/`. The string `bmm-workflow-status.yaml` does not appear anywhere in `src/`.
  - 4 completion steps carry generic "update workflow status (if exists)" prose — no registry path resolution, no template auto-create, no fallback chain.
  - Cross-cuts E4: every "if exists" in completion steps is the same `do X (if Y)` defect class.
  - **In-the-wild registry instances exist with substantial schema drift** — see §5.2 (RS-1..RS-8).
- **E4 — Dev-story workflow defect:**
  - Slash-command was patched outside this repo (`~/.claude/commands/bmad/dev-story.md`).
  - **Canonical skill workflow `src/bmm-skills/4-implementation/bmad-dev-story/workflow.md` STILL EXHIBITS THE STRUCTURAL DEFECT:** Step 1 HALTs on missing story file; Step 10 only updates internal sections of an assumed-existing file. Slash-command patch did not cascade into the skill.

**FSM-in-prose anti-pattern, corpus-wide evidence (`S1` smoking gun):**
- Preamble "**LOAD the FULL workflow.md, READ its entire contents and follow its directions exactly!**" appears across **8 projects** in episodic memory: claudefat (2026-03-05), describe (2026-03-07/08), linkml-lkml--bmad (2026-03-03/04/05), archbox-docs (2026-03-06), archbox (2026-04-03), linkml (2026-03-21, 2026-04-17), padimae (2026-02-24).
- Two preamble variants: **workflow-level** ("LOAD the FULL {workflow.md}…") and **task-level** (`<steps CRITICAL="TRUE"> 1. Always LOAD the FULL {core/tasks}…`).
- Emphatic-prose-with-CRITICAL-banner = BMAD's own admission that procedural prose is unreliable substrate.
- Per A7: "LLMs are very poor state machines (i.e. a non-deterministic state machine == gambling)" — the verbose ceremonial preambles are heritage from less capable LLMs. Replace prose-FSM with real deterministic FSM where LLM is operator/facilitator, not state-keeper.
- Direct from bmad-master 2026-04-01 (`25266018...jsonl`): "BMAD is instructions. Claude is the engine. There is no separate BMAD runtime that can 'assert control' — when you invoke a BMAD workflow, Claude reads the workflow.xml and instructions, and it's Claude's discipline (or lack of it) that determines whether the protocol is followed."

### 1.3 Project corpus inventory (BG-2 full report: `01-project-inventory.md`, 155 lines)

**All 9 BMAD projects are pristine** when measured by the **correct anchor** — `_bmad-output/` directory's **creation epoch** (`stat %W` birthtime; ctime fallback), NOT the newest artifact mtime inside it. Output artifacts are written constantly during normal use; mtime-on-newest is wrong (Claude got this wrong initially; corrected).

**Customization lives outside `_bmad/` proper** — in peripheral output dirs (`liaison/`, `failings/`, `learnings/`) and in promoted Python packages (now via `lkml`, not deprecated `lkml-gen`).

| Project | Customization | Substrate-tier evidence |
|---|---|---|
| **linkml** | High | E2 already applied via tracked commit `d5dbbdb0a`; `liaison/` + `failings/` + `learnings/`; the two BMAD analysis docs (1434 + 632 lines); promoted Jinja2/Mustache adoc templates |
| **architecture** | High | Owns the AsciiDoc→YAML toolchain (parsers + extract-section.sh + .adoc-index + extracted/*.yaml + fw/extracted/*.yaml); F1/D1/D3 promotion candidates |
| **archbox** | Medium | `liaison/` mailbox identical to linkml's; custom `_bmad-output/architecture/` doc layout |
| Factweave | Low | Content-only, no toolchain |
| padimae, describe, claudefat, teague, relloq | None | Pristine |

**Note:** BMAD-METHOD repo itself is **substrate, not consumer** — exclude from "BMAD-using project" analysis.

**Two BMAD install versions in the wild:**
- Older naming (architecture, Factweave): `bmad-create-product-brief`, `bmad-brainstorming` (workflow-prefixed).
- Newer naming (archbox, linkml): unprefixed.
- Both carry `step-01b-continue.md`. Substrate has been moving — install-version awareness is a Wave 0 / governance concern.

**Zero ad-hoc workarounds** (`WORKAROUND`/`HACK`/`TODO(BMAD)`/`FIXME`/`HOTFIX`) in inspected `_bmad-output/` trees — but per user 2026-04-29: **"the workarounds were the adhoc extraction of YAML from asciidoc with the scripts."** The workaround surface lives **outside** `_bmad-output/`: extraction infrastructure is the workaround. Promotion candidate: **deprecate, do not delete** (per A4 + A8).

### 1.4 User answers Q3-Q12 (verbatim, 05-open-questions.md)

**A3 (state-tracking substrate):** **(a)** Schema-validated YAML registry as source of truth. Workflows write through a validator. Step files become pure renderers against the registry. Markdown `stepsCompleted` removed; XML `<step n="N">` ordinals removed. State is data, not control flow.

**A4 (Python rewrite scope):** **(e)** A LinkML metamodel for each YAML schema used to generate an adoc, plus a lkml gen template. (`lkml-gen` is no longer a tool, all CLI capabilities accessed via `lkml`.) Defer (or eliminate) extraction with a different flow: **proposed YAML generates adoc for review, changes made to proposed YAML, then adjudicated.**

**A5 (rebase strategy):** **(a)** Rebase enhance onto main.

**A6 (file-name reconciliation):** Origin of renames in architecture skill unknown — may have been a Claude artifact. Ride upstream naming; map Evolution-1's INTENT onto whatever the file name happens to be. Evolution-1 fix files become historical evidence, not patch-ready content.

**A7 (FSM remediation scope):** Incremental — **(a) workflow step files first, then (c) pilot agent activation.** Not committed to BMAD. Verbose ceremonial preambles are heritage from less capable LLMs. Likely the bare-bones steps are much more simple and straightforward with verifiable gate tests and FSM logic implemented as a proper state machine. Hard to separate annoying ceremonial aspects from extra decorations required for behavior. **"LLMs are very poor state machines (i.e. a non-deterministic state machine == gambling)."**

**A8 (AsciiDoc + YAML + scripts promotion):** Migration path required. Cannot abandon extraction scripts but can deprecate — use in case of authoritative adocs. YAML as exclusively authoritative is **not yet feasible (pragmatically)**. Most BMAD users probably unfamiliar with AsciiDoc and don't care about the rigor, but **serious projects need the features it offers**. **"If I continue with BMAD, it will likely be a custom implementation, because the integration with IDEs and dashboards are terrible, no doubt exacerbated by the limitations exposed in this study."** ← strategic threat.

**A9 (better-ADR-capture):** **Yes** — better ADR capture is the headline deliverable for the branch. Implies: (a) ADR canonical shape (Status / Context / Decision / Consequences / Rationale / Links / Alternatives); (b) YAML schema includes `rationale` field as REQUIRED; (c) `architecture-decision-template.md` updated; (d) parsers extract rationale.

**A10 (review-edit-extract-diff loop promotion):** **(e)** Agile process consumes available material. **Markdown, YAML, or AsciiDoc enter the BMAD system as authoritative; system normalizes and curates ground truth; system generates derivatives to facilitate change proposals, review, and revision. Once accepted, YAML remains the ground truth; change proposals and review derivatives generated/approved.** YAML is easier to validate and version-manage at the business object level; formatted texts harder to mechanistically govern and align. **Use of PAV to document the relationship should be normative.**

**A11 (FR-extraction substrate scope):** **Must begin with a project-level initialization and GOVERNANCE of a LinkML-compatible document (YAML/JSON/LinkML), with authoritative sources clearly specified, and meticulously curated, none of this slop dropping files wherever, and then languishing.** Required regardless of whether it is BMAD, GSD, or pro-workflow.

**A12 (registry + requirements unification):** **Do NOT get too locked into a unified LinkML metamodel covering registry + requirements. The issue is broader than requirements. The AND is signal that composition may be indicated. Why is one better than multiple? Are you mixing concerns?** Drift occurred because the user was tired of explaining to Claude/BMAD why their convention was deficient — preferred method for requirement identifiers is **enterprise-wide unique and hierarchical**. Good requirements engineering does not depend on the project. **The schema used in elt and fw are probably the closest to ideal, but the CSV generated for review was useful. A LinkML model that exposed the spreadsheet column names mapped to the YAML schema, while preserving the fidelity of the asciidoc source is likely to be the big win.** **The BIG win comes from traceability between the derived artifacts.** A project-level document (LinkML) should specify the identifier schema (present in elt and fw, but semantically weaker than it should be). If combined with **CALM or C4 Architecture artifact (recall CALM metamodel work)**, this creates traceability across **CONOPS, requirements, user stories, use cases, and architecture**. Similar discipline for **quality assurance, prompt engineering (BAML), and other lkml 'expansion packs'** and consistent use of **PAV discipline** would link lifecycle artifacts.

---

## §2 The Right Data Flow

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PROJECT-LEVEL GOVERNANCE                         │
│  governance.yaml (LinkML-validated)                                 │
│   • identifier_schema (enterprise-wide unique + hierarchical)       │
│   • authoritative_sources (paths + format per artifact type)        │
│   • format_policy (md/yaml/adoc accepted; YAML canonical)           │
│   • pav_discipline (normative)                                      │
│   • curation_locations                                              │
└────────────────┬────────────────────────────────────────────────────┘
                 │ binds all artifacts below
                 ▼
┌─────────────────────────────────────────────────────────────────────┐
│         AUTHORITATIVE SOURCE (any of: md / yaml / adoc)             │
│  Operator authors here. AsciiDoc preferred for serious technical    │
│  work. YAML co-equal. Markdown supported, discouraged for technical │
│  artifacts.                                                          │
└────────────────┬────────────────────────────────────────────────────┘
                 │ normalize (Python parsers, lkml templates)
                 ▼
┌─────────────────────────────────────────────────────────────────────┐
│      YAML GROUND TRUTH (LinkML-validated, per-artifact-type)        │
│  Decoupled schemas (composition, NOT unification per A12):          │
│    • bmm-workflow-status.linkml.yaml (registry)                     │
│    • requirement.linkml.yaml                                        │
│    • use-case.linkml.yaml                                           │
│    • user-story.linkml.yaml                                         │
│    • subsystem.linkml.yaml                                          │
│    • data-model.linkml.yaml                                         │
│    • acceptance-criterion.linkml.yaml                               │
│    • architecture-decision.linkml.yaml  (A9 headline)               │
│  Bound by: governance.yaml + PAV traceability + optional CALM/C4    │
│  Expansion packs: QA, BAML prompt-engineering, etc.                 │
└────────────────┬────────────────────────────────────────────────────┘
                 │
       ┌─────────┼─────────┐
       │         │         │
       ▼         ▼         ▼
   ┌────────┐ ┌──────┐ ┌──────────┐
   │ ADOC   │ │  MD  │ │ CSV      │  ← derivatives via lkml gen templates
   │ (rev.) │ │(rev.)│ │ (review) │     (Jinja2/Mustache)
   └────┬───┘ └──┬───┘ └────┬─────┘     Spreadsheet review = "useful"
        │       │           │           per A12
        └───────┴───────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────────────┐
│           REVIEW-EDIT-EXTRACT-DIFF LOOP (per A10/Q10-e)             │
│  1. Operator edits source (or proposed-YAML in inverted-flow case)  │
│  2. Agent extracts/normalizes YAML                                  │
│  3. System diffs new YAML against prior committed YAML              │
│  4. Operator reviews diff = change-proposal artifact (≡ PR diff)    │
│  5. Accept → commit; reject → re-edit cycle                         │
│                                                                     │
│  PAV traceability links derived ↔ source at every step.             │
└─────────────────────────────────────────────────────────────────────┘
```

**Inverted-flow case (per A4-e):** instead of `adoc → extract YAML → derived`, the new direction is `proposed YAML → generate adoc for review → adjudicate changes → YAML ground truth`. This eliminates extraction in the proposed-YAML case. Extraction scripts retained as legacy import for case of authoritative adocs (per A8 migration discipline).

**Spreadsheet derivative (per A12 "the big win"):** LinkML model exposes spreadsheet column names mapped to YAML schema, while preserving fidelity of AsciiDoc source. Bridge between three audiences: spreadsheet-reviewers (human), YAML (machine), adoc (engineering rigor).

**CALM/C4 metamodel integration (per A12):** combine LinkML schemas with CALM or C4 Architecture artifact for traceability across CONOPS / requirements / user stories / use cases / architecture. Optional but high-value. **Open question — see §8.**

---

## §3 The Right Components

### 3.1 PROMOTE upstream (existing, with refinement)

| Component | Source | Refinement | Evidence |
|---|---|---|---|
| `parse-uc.awk`, `parse-us.awk`, `parse-ss.awk`, `parse-dm.awk` (524 lines) | `~/repos/architecture/tools/parsers/` | Rewrite in Python per A4-e + A8; package as `bmad-extract` toolset upstream | Q4-A4-e |
| `extract-section.sh` (bash) | `~/repos/architecture/elt/` | Deprecated alongside Python rewrite. **Retained for case of authoritative adocs** per A8 | Q8-A8 |
| Adoc-generation Jinja2/Mustache templates | `lkml` tool's templates | Consumed by canonical schema definitions; one template per schema | A4-e |
| `liaison/` mailbox protocol | linkml + archbox | New skill `bmad-liaison` for cross-project coordination — A4-category candidate | BG-2, §1.3 |
| `failings/` and `learnings/` peripheral output dirs | linkml | Document as canonical output convention (not just linkml-specific) | BG-2 |
| **G/I/C scoring rubric + contribution-readiness %** | linkml's two analysis docs (1434 + 632 lines) | **Adopt as-is**; do NOT re-derive a parallel taxonomy | §1.1 |
| **Review-edit-extract-diff loop** | architecture repo's adoc↔YAML pattern | Codify as canonical pattern (skill or convention extending `bmad-correct-course`) | Q10-A10-e |
| **PAV discipline** | linkml memory entry "PAV-first traceability" commit `6c552e9` | Normative across all derived↔authoritative relationships | A10 |

### 3.2 DEPRECATE (parallel-operation period, then sunset, per A8)

| What | Why | Migration |
|---|---|---|
| All `lkml-gen` references | Tool no longer exists; CLI capability via `lkml` per A4 correction | Find/replace + update docs |
| Bash `extract-section.sh` as primary path | Replaced by Python parsers; retained as fallback for authoritative adocs | Python rewrite ships first; bash sunsets after parallel period |
| `<step n="N">` ordinals in `instructions.xml` (FSM-as-prose) | A3 = pure renderers against schema-validated YAML registry | Strip from step files as Wave 2 ships; replaced by validator-driven state |
| Markdown `stepsCompleted` frontmatter | A3 = state lives in registry, not in document frontmatter | Migrate to registry; leave doc unchanged |
| "LOAD the FULL workflow.md, READ its entire contents…" preambles | Heritage from less-capable LLMs per A7; substrate is wrong | Replace with deterministic FSM where LLM is operator/facilitator |
| `FR-001`-style restart-per-document numbering | "ROOKIE move" per Factweave 2026-03-24; ENCOSE-violation | Identifier schema in governance.yaml; subsystem-prefixed hierarchical IDs |

### 3.3 BUILD new (skills, schemas, workflows)

**Skills:**
| Skill | Purpose | Scaffolding |
|---|---|---|
| `bmad-init-governance` | Generates `governance.yaml` at project root: identifier_schema, authoritative_sources, format_policy, pav_discipline, curation_locations | Use `bmad-builder:bmad-workflow-builder` |
| `bmad-validate-governance` | Checks PAV records exist, identifiers conform to schema, no orphan artifacts | Same |
| `bmad-liaison` | Cross-project coordination mailbox protocol (promoted from linkml + archbox) | Same |
| `bmad-extract` toolset | Python parsers for adoc → YAML extraction (UC/US/SS/DM/AC/ConOps) | Promoted from `~/repos/architecture/tools/parsers/` |
| `bmad-derive` toolset | YAML → adoc/md/CSV generation via lkml gen templates | Promoted from lkml tool's templates |
| `bmad-review-extract-diff` (or extension to `bmad-correct-course`) | Wraps the Q10-A10-e loop: extract → diff vs committed → present diff → accept/reject/iterate | Skill |

**Workflow patches (existing skills):**
| Where | What | Source |
|---|---|---|
| `src/bmm-skills/4-implementation/bmad-dev-story/workflow.md` | Step 1: create story file if missing (currently HALTs); Step 10: ensure all sections written, not just internal updates | E4 verdict |
| `src/bmm-skills/.../bmad-correct-course/` | Add full E1 marker pattern (FACILITATOR / A/P/C / stepsCompleted / FORBIDDEN-load-next / NEVER-generate-without-input) | E1 verdict + `fixes/evolution-1/checklist.md` |
| `src/bmm-skills/.../bmad-check-implementation-readiness/` | Same E1 pattern across all 6 step files (zero markers currently) | E1 verdict + `fixes/evolution-1/step-{02,06}-*.md` |
| `bmad-create-epics-and-stories/` | Add explicit `State Recovery Check` section to workflow.md + `step-01b-continue.md` + step-01 Section-0 guard | E2 P0 target + linkml Cluster 0 |
| `bmad-check-implementation-readiness/`, `bmad-generate-project-context/`, `bmad-sprint-planning/`, `bmad-create-story/` | Same E2 pattern | E2 P1/P2 targets |
| ALL skills with step-01b-continue.md but no workflow.md gate | Add `State Recovery Check` section to workflow.md | E2 verdict |

**Schemas (LinkML):**
| Schema | Required fields | Notes |
|---|---|---|
| `bmm-workflow-status.linkml.yaml` | `id` (prefixed), `expected` (enum: required/recommended/optional/conditional/skipped), `state` (enum), `output` (path or list per RS-1), `phase`, `description`, `last_updated`, `rationale` (FR-3/RS-6), `provenance` (PAV per A10), `schema_version`, `output_version` | Multi-output via list-valued `output`; backward-compat read script |
| `requirement.linkml.yaml` | `id` (governed format per governance.yaml), `title`, `rationale` (REQUIRED per FR-3 + Q9), `traces_to[]` (typed PAV), `version`, `phase`, `strength`, `source`, `test_procedure[]`, `subsystem` (FR-2), `capability_area` (FR-7) | ELT/FW as base, semantically strengthened |
| `use-case.linkml.yaml` | Mirror requirement structure with UC-specific fields | Same |
| `user-story.linkml.yaml` | Same + `acceptance_criteria[]` with `version` per AC + artifact-level `version` (FR-8) | Versioned ACs already in fw/extracted |
| `subsystem.linkml.yaml` | + `includes[]` with provenance preservation (FR-5) | |
| `data-model.linkml.yaml` | Mirror | |
| `acceptance-criterion.linkml.yaml` | Mirror | |
| `architecture-decision.linkml.yaml` (A9 headline) | `id`, `status` enum, `context`, `decision`, `consequences[]`, `rationale` (REQUIRED, the headline fix), `alternatives[]`, `links[]`, PAV | Branch's headline deliverable |

**Schema binding (per A12 composition):**
- Each schema `id` validated against `governance.yaml`'s `identifier_schema`.
- Cross-references via `traces_to[]` (typed PAV) — same shape across all schemas (FR-4).
- No master metamodel that subsumes all schemas. Composition over unification.
- CALM/C4 metamodel integration optional, recommended (open question §8).
- Expansion-pack pattern for QA, BAML, etc. (per A12).

### 3.4 Schemas already evidence-based (preserve, do not redesign)

- ELT/FW user-story schema with versioned `acceptance_criteria[].version` (per `fw/extracted/fwa-user-stories.yaml`) — keep this structure.
- ID convention `{TYPE}-{SYSTEM}-{NN}` from Factweave 2026-03-24 — keep, generalize to enterprise-wide-unique-hierarchical per A12.
- User's proposed full requirement schema (`source`, `phase`, `strength`, `rationale`, `traces_to`, `test_procedure`) — implement as REQUIRED in `requirement.linkml.yaml`; the existing implementation lost these fields, regaining them is FR-3 closure.

---

## §4 The Right Moves (action set, with dependency graph)

Wave numbering from prior plan is **dropped**. Moves are organized by topology of dependencies.

### Move catalogue

| ID | Move | Depends on | Approval required | Blast radius |
|---|---|---|---|---|
| **M1** | Rebase `enhance/better-adr-capture` onto `main` | (none) | Yes — destructive history op | This branch only |
| **M2** | Bootstrap project governance scaffolding spec (`governance.yaml` schema + skill design) | M1 | Yes — substrate decision | New skill, no existing skill modified |
| **M3** | Define `architecture-decision.linkml.yaml` schema (A9 headline) | M2 | Yes — per-schema decision | New schema only |
| **M4** | Patch canonical `bmad-dev-story/workflow.md` for E4 (Step 1 create-if-missing, Step 10 full-write) | M1 | Yes — workflow change | Single skill workflow |
| **M5** | Patch `bmad-correct-course/` and `bmad-check-implementation-readiness/` with E1 markers | M1 | Yes — workflow change | Two skills |
| **M6** | Patch `bmad-create-epics-and-stories/` for E2 (workflow.md State Recovery Check + step-01b-continue.md + step-01 Section-0 guard) | M1 | Yes — workflow change | Single skill |
| **M7** | Define `bmm-workflow-status.linkml.yaml` schema (E3 substrate, RS-1..RS-8) | M2 | Yes — per-schema decision | New schema |
| **M8** | Build Python migration script: existing `bmm-workflow-status.yaml` instances → new schema (4 in-the-wild instances: luka, teague, linkml, kai/Elements) | M7 | Yes — touches consumer projects | Migration utility, parallel-op |
| **M9** | Wire up E3 writeback machinery against new schema in completion steps of multi-step workflows (replace "if exists" prose with registry path resolution + template auto-create) | M7, M8 | Yes — workflow changes | All multi-step workflows |
| **M10** | Define remaining LinkML schemas: `requirement`, `use-case`, `user-story`, `subsystem`, `data-model`, `acceptance-criterion` (FR-1..FR-8 closure) | M2 | Yes — substrate set | New schemas, no skill changes yet |
| **M11** | Promote `~/repos/architecture/tools/parsers/parse-{uc,us,ss,dm}.awk` as Python `bmad-extract` toolset upstream | M10 | Yes — new tool surface | New CLI tool |
| **M12** | Promote `lkml` tool's adoc-generation templates as canonical `bmad-derive` upstream | M10, M11 | Yes — new tool surface | New CLI tool |
| **M13** | Build `bmad-init-governance` + `bmad-validate-governance` skills | M2 | Yes — new skill surface | Two new skills |
| **M14** | Build `bmad-liaison` skill from linkml/archbox `liaison/` mailbox pattern | M2 | Yes — new skill | New skill |
| **M15** | Build `bmad-review-extract-diff` skill (or extend `bmad-correct-course`) for Q10-A10-e loop | M11, M12 | Yes — new skill | New skill or extension |
| **M16** | Patch all skills with step-01b-continue.md but no workflow.md gate to add explicit `State Recovery Check` section (E2 across-the-board) | M1 | Yes — workflow changes | All applicable skills |
| **M17** | Apply E2 pattern to remaining P1/P2 targets: `bmad-check-implementation-readiness`, `bmad-generate-project-context`, `bmad-sprint-planning`, `bmad-create-story` | M16 | Yes — workflow changes | Four skills |
| **M18** | Pilot real FSM: replace prose-FSM in ONE workflow's step files with deterministic state machine spec (per A7-a). Workflow choice: TBD — recommend `bmad-create-architecture` (most-exercised in user projects) | M9, after M5/M6/M16/M17 stabilize | Yes — substrate experiment | One skill, isolated |
| **M19** | Pilot agent activation FSM (per A7-c): one agent, isolated, after M18 lands. Recommend `bmad-master`. | M18 stable | Yes — substrate experiment | One agent |
| **M20** | Define ENCOSE-aligned identifier schema enforcement in `bmad-validate-governance` (subsystem-prefixed, hierarchical, enterprise-wide unique) | M13 | Yes — governance addition | Validator |
| **M21** | CSV review-derivative generation (per A12 "big win"): LinkML model maps spreadsheet column names ↔ YAML schema while preserving adoc fidelity | M12 | Yes — derivative tooling | New derivation rule |
| **M22** | Optional CALM/C4 metamodel integration research + binding | M20 | Yes — substrate addition | Open question §8 |
| **M23** | Expansion-pack pattern: LinkML schemas for QA artifacts, BAML prompt-engineering | M22 (or M10 if standalone) | Yes — schema additions | New schemas |
| **M24** | Hygiene pass: dedup, install-version awareness in `bmad-validate-governance`, gitignore patterns | After M16/M17 stabilize | Yes — but mostly mechanical | Across repo |
| **M25** | Decide branch strategy: continue on `enhance/better-adr-capture` for upstreaming OR fork branch for custom implementation per A8 strategic threat | (none — strategic) | **User-only decision** | Branch scope |

### Dependency graph (textual)

```
M25 (strategic, parallel) ─────────────────────────────┐
                                                       │
M1 (rebase) ─┬─ M2 (governance spec) ─┬─ M3 (ADR schema, A9)
             │                        ├─ M7 (registry schema) ─ M8 (migration) ─ M9 (E3 writeback)
             │                        ├─ M10 (5 LinkML schemas) ─┬─ M11 (parsers) ─┬─ M12 (templates)
             │                        ├─ M13 (governance skills) │                 │
             │                        │                          │                 ├─ M21 (CSV derivative)
             │                        ├─ M14 (liaison skill)     │                 ├─ M15 (review-extract-diff)
             │                        └─ M20 (ENCOSE validator)
             │                                                   M18 (FSM pilot workflow) ─ M19 (FSM pilot agent)
             ├─ M4 (E4 dev-story patch)                          ↑
             ├─ M5 (E1 patches)                                  │
             ├─ M6 (E2 epics-and-stories)        ────────────────┤ (after E1/E2/E3/E4 stabilize)
             ├─ M16 (E2 across-the-board)        ────────────────┤
             └─ M17 (E2 P1/P2 patches)           ────────────────┘

M22 (CALM/C4) ─ M23 (expansion packs)  ← optional, post-core-stable
M24 (hygiene)                          ← post-core-stable
```

### Per-move detail (terse, structured for ultra-plan ingestion)

For each move, ultra-plan should produce: (a) single concrete first-step, (b) what would invalidate, (c) explicit user approval point.

**M1 — Rebase**
- WHAT: `git rebase main` on `enhance/better-adr-capture`; resolve conflicts; verify 4 enhance commits replay cleanly.
- WHY: A5 + main has moved 28+ commits including `9725b0ae refactor(skill)` flatten, `25d24d02` dev-story to native skill, `0380656d refactor: consolidate agents into phase-based skill directories #2050`. All evolution work targets must use post-rebase paths.
- INVALIDATES: massive conflicts unresolvable without scope expansion → fall back to merge (Q5-(b)) with user approval.
- APPROVAL: yes, before rebase starts (destructive history op).

**M2 — Governance spec**
- WHAT: Author `governance.yaml` LinkML schema. Fields: `identifier_schema` (with required namespace declaration), `authoritative_sources[]` (path + format per artifact type), `format_policy` (md/yaml/adoc accepted; YAML canonical per A10), `pav_discipline` (normative per A10), `curation_locations[]`.
- WHY: A11 — required regardless of BMAD/GSD/pro-workflow.
- INVALIDATES: none — this is a precondition to all downstream substrate work.
- APPROVAL: yes, on the schema fields.

**M3 — ADR schema (A9 headline)**
- WHAT: `architecture-decision.linkml.yaml` — `id`, `status` enum, `context`, `decision`, `consequences[]`, `rationale` REQUIRED, `alternatives[]`, `links[]`, PAV.
- WHY: A9 explicit — the branch's reason to exist. Closes the lost-rationale failure surface (FR-3 / D2).
- INVALIDATES: none.
- APPROVAL: yes, on schema field set.

**M4 — Dev-story Step 1/10 patch**
- WHAT: Patch `src/bmm-skills/4-implementation/bmad-dev-story/workflow.md`: Step 1 creates story file if missing; Step 10 ensures all sections written, not just internal updates.
- WHY: E4 verdict — defect persists in canonical workflow despite slash-command patch.
- INVALIDATES: if upstream has already addressed (must verify post-rebase).
- APPROVAL: yes, after rebase, before patch commit.

**M5 — E1 patches (correct-course + check-implementation-readiness)**
- WHAT: Add full 5-marker pattern (FACILITATOR / A/P/C / stepsCompleted / FORBIDDEN-load-next / NEVER-generate-without-input) to `bmad-correct-course/` and all 6 step files of `bmad-check-implementation-readiness/`.
- WHY: E1 verdict — these have ZERO markers and are precisely where the original Evolution-1 fix files were authored for.
- INVALIDATES: upstream may have addressed; verify post-rebase.
- APPROVAL: yes, on retargeted file content.

**M6 — E2 epics-and-stories**
- WHAT: Patch `bmad-create-epics-and-stories/` with workflow.md `State Recovery Check` section + `step-01b-continue.md` + step-01 Section-0 guard.
- WHY: E2 P0 target. linkml Cluster 0 is essentially this generalized.
- INVALIDATES: same.
- APPROVAL: yes, on patches.

**M7 — Registry schema**
- WHAT: `bmm-workflow-status.linkml.yaml`. Closes RS-1..RS-8.
- WHY: kai/Elements broke the schema; 4 in-the-wild instances drift; E3 spec unimplemented.
- INVALIDATES: if A12-style composition reveals the registry needs different shape than per-artifact schemas; surface for clarification.
- APPROVAL: yes.

**M8 — Migration script**
- WHAT: Python script: existing `bmm-workflow-status.yaml` instances → new schema. Targets: `luka/docs/`, `teague/docs/`, `linkml/docs/`, `kai/Plans/Elements/`. Backward-compatible read.
- WHY: A8 migration discipline — deprecate-don't-delete.
- INVALIDATES: if any project's schema is too divergent to migrate (kai/Elements is closest); fall back to per-project adapter.
- APPROVAL: yes, before running against any consumer project.

**M9 — E3 writeback machinery**
- WHAT: Replace "update workflow status (if exists)" prose in completion steps with registry path resolution + template auto-create + fallback chain.
- WHY: cross-cuts E4; every "if exists" is the same defect class.
- INVALIDATES: if registry schema migrates incompletely (M8 gap).
- APPROVAL: yes, per workflow patched.

**M10 — Five remaining LinkML schemas**
- WHAT: `requirement.linkml.yaml`, `use-case.linkml.yaml`, `user-story.linkml.yaml`, `subsystem.linkml.yaml`, `data-model.linkml.yaml`, `acceptance-criterion.linkml.yaml`. ELT/FW as base, semantically strengthened (FR-1..FR-8 closure).
- WHY: Q11/A11 — substrate must close all 8 deficiencies in coherent set.
- INVALIDATES: if A12 reveals additional artifact types should be in the set first (CONOPS? capability? journey?); surface for clarification.
- APPROVAL: yes, on full schema set as a unit.

**M11 — Python parser promotion**
- WHAT: Rewrite 4 awk parsers as Python; package as `bmad-extract` toolset.
- WHY: A4-e (Python rewrite) + A8 (deprecate awk in parallel-op).
- INVALIDATES: if Python rewrite cannot match awk fidelity in edge cases; surface specific divergences for user.
- APPROVAL: yes, on output equivalence test results.

**M12 — Template promotion**
- WHAT: Promote `lkml` tool's Jinja2/Mustache adoc-generation templates as canonical `bmad-derive` upstream toolset.
- WHY: A4-e — one template per schema.
- INVALIDATES: if templates are too project-specific; refactor to schema-driven first.
- APPROVAL: yes.

**M13 — Governance skills**
- WHAT: `bmad-init-governance` (generates governance.yaml) + `bmad-validate-governance` (PAV/identifier/orphan checks).
- WHY: A11 — governance is prerequisite to everything else.
- INVALIDATES: if `bmad-builder:bmad-workflow-builder` produces non-conformant skill structure; iterate.
- APPROVAL: yes.

**M14 — Liaison skill**
- WHAT: `bmad-liaison` from linkml + archbox `liaison/` mailbox pattern.
- WHY: cross-project coordination primitive observed in two projects.
- INVALIDATES: if pattern doesn't generalize beyond linkml/archbox use case.
- APPROVAL: yes — and per A8 may DEFER to follow-on if scope is too speculative.

**M15 — Review-extract-diff skill**
- WHAT: `bmad-review-extract-diff` (or extension to `bmad-correct-course`) wrapping the Q10-A10-e loop.
- WHY: A10 — agile multi-format-entry process is normative.
- INVALIDATES: if the diff surface is incompatible with operator review tooling (IDE/dashboard concern per A8).
- APPROVAL: yes.

**M16 — E2 across-the-board**
- WHAT: Add `State Recovery Check` section to workflow.md in all skills that have step-01b-continue.md but no gate.
- WHY: E2 verdict — gate not authoritative even where step-01b exists.
- INVALIDATES: upstream may have addressed.
- APPROVAL: yes per skill.

**M17 — E2 P1/P2 patches**
- WHAT: E2 pattern applied to `bmad-check-implementation-readiness`, `bmad-generate-project-context`, `bmad-sprint-planning`, `bmad-create-story`.
- WHY: E2 verdict — none carry step-01b file.
- INVALIDATES: same.
- APPROVAL: yes per skill.

**M18 — FSM pilot workflow**
- WHAT: Replace prose-FSM with deterministic state machine in step files of ONE workflow. Recommend `bmad-create-architecture` per A7-a.
- WHY: A7 — incremental. Verifiable gate tests + proper FSM logic. LLM is operator/facilitator.
- INVALIDATES: if FSM substrate doesn't simplify (or actively complicates) the workflow; user veto.
- APPROVAL: yes — and explicit approval required because A7 says ceremonial-prose simplification is itself a research question. **No binary success criteria.** **No "≤ 50% token threshold" or "no behavioral regression."** Outcomes reported multi-dimensionally for user adjudication.

**M19 — FSM pilot agent**
- WHAT: A7-c — pilot agent activation FSM. Recommend `bmad-master`.
- WHY: A7 incremental progression after M18 stable.
- INVALIDATES: if M18 outcome is negative.
- APPROVAL: yes, gate on M18 outcome user-adjudicated.

**M20 — ENCOSE validator**
- WHAT: `bmad-validate-governance` enforces subsystem-prefixed, hierarchical, enterprise-wide unique IDs.
- WHY: A12 — good requirements engineering does not depend on the project.
- INVALIDATES: if existing identifier schemas in active projects can't migrate; deprecation path needed.
- APPROVAL: yes.

**M21 — CSV review derivative**
- WHAT: Per A12 "big win" — LinkML model maps spreadsheet column names ↔ YAML schema, preserving adoc fidelity.
- WHY: A12 — bridges three audiences.
- INVALIDATES: none — additive.
- APPROVAL: yes on column-name mapping spec.

**M22 — CALM/C4 metamodel (open)**
- WHAT: Research existing CALM metamodel work; design integration with LinkML schemas for CONOPS / requirements / US / UC / architecture traceability.
- WHY: A12 — "the BIG win comes from traceability between the derived artifacts."
- INVALIDATES: see §8 open questions.
- APPROVAL: yes, after research.

**M23 — Expansion packs**
- WHAT: LinkML schemas for QA artifacts, BAML prompt-engineering, plus framework for new packs.
- WHY: A12 — "expansion packs" pattern.
- INVALIDATES: if pack pattern doesn't cleanly compose with core.
- APPROVAL: yes per pack.

**M24 — Hygiene**
- WHAT: Dedup, install-version awareness, gitignore patterns.
- WHY: BG-2 noted two BMAD install versions in the wild; substrate has been moving.
- INVALIDATES: minor.
- APPROVAL: yes, but mostly mechanical.

**M25 — Branch strategy decision**
- WHAT: Decide: continue on `enhance/better-adr-capture` for upstream contribution OR fork branch for user's custom implementation.
- WHY: A8 — "If I continue with BMAD, it will likely be a custom implementation, because the integration with IDEs and dashboards are terrible."
- INVALIDATES: not Claude's call.
- APPROVAL: **user-only decision**. Strategic, not technical. May change scope of M3-M24 substantially.

---

## §5 Substrate Defects to Address (the deficiency catalogues)

### 5.1 FR-tier — requirements YAML deficiencies (8 total, per FR extraction)

| # | Deficiency | Closes via |
|---|---|---|
| FR-1 | BMAD restarts requirement numbering at -001 per document | M2 (governance.yaml identifier_schema) + M20 (validator) |
| FR-2 | No subsystem identifier in requirement IDs | Same |
| FR-3 | Schema lost `rationale`, `source`, `traces_to`, `phase`, `strength`, `test_procedure` despite proposed schema | M3 (ADR), M10 (other schemas) |
| FR-4 | Cross-reference mechanics (`<<file.adoc#{idprefix}ID,display [ID]>>`) no equivalent in markdown | M10 (`traces_to[]` typed PAV uniform across schemas) |
| FR-5 | "Includes" provenance lines (`Includes VM-US-001, …`) lost in YAML extraction | M10 (`subsystem.linkml.yaml` `includes[]` with provenance) + M11 (Python parser captures) |
| FR-6 | No traceability matrix from FR → UC → US → DM → test procedure | M22 (CALM/C4 integration) — primary; or M10 (`traces_to[]` traversal) |
| FR-7 | Capability area / subsystem hierarchy implicit in adoc, not explicit in YAML | M10 (`capability_area`, `subsystem` as first-class fields) |
| FR-8 | Versioning of acceptance criteria implemented per-AC; no version metadata at artifact level | M10 (`version` at artifact-level, in addition to per-AC) |

### 5.2 RS-tier — registry schema deficiencies (8 total)

| # | Deficiency | Observed in | Closes via |
|---|---|---|---|
| RS-1 | Schema cannot represent multi-output workflows | kai/Elements vs others | M7 (`output` list-valued) |
| RS-2 | `status` field overloaded (expected/state/output mixed) | All 4 wild | M7 (split: `expected`, `state`, `output`) |
| RS-3 | No `create-epics-and-stories` entry in luka, teague | luka, teague | M6 + M9 (writeback machinery creates) |
| RS-4 | Field-set drift (`command`, `notes`, `last_workflow_date`, `brainstorming` vary) | All 4 | M7 (canonical schema) + M8 (migration) |
| RS-5 | No subsystem prefix on `name` | All 4 | M7 + M20 (validator) |
| RS-6 | No `rationale`, `traces_to`, version metadata at workflow-entry level | All 4 | M7 |
| RS-7 | No diff-friendly canonical form (varying key order, optional fields, comments) | All 4 | M7 (deterministic canonical form) |
| RS-8 | Project rename drift not handled (teague's `project_name: "genmap"`) | teague | M2 (governance project naming) + M8 (migration) |

### 5.3 Evolution gaps

| Evolution | Gap | Closes via |
|---|---|---|
| E1 | `bmad-correct-course/` zero markers; `bmad-check-implementation-readiness/` zero markers | M5 |
| E2 | No workflow.md `State Recovery Check` section anywhere; 5 P0/P1/P2 targets carry no step-01b | M6 + M16 + M17 |
| E3 | Spec unimplemented in src/; in-the-wild instances drifted | M7 + M8 + M9 |
| E4 | Canonical `bmad-dev-story/workflow.md` defect persists | M4 |

---

## §6 Constraints / Invariants (bind every move)

| # | Invariant | Source |
|---|---|---|
| C1 | Project-level governance scaffolding is prerequisite to ALL substrate work — required regardless of BMAD/GSD/pro-workflow | A11 |
| C2 | Identifier schema = enterprise-wide unique + hierarchical | A12 |
| C3 | PAV discipline is normative for every derived↔authoritative relationship | A10 |
| C4 | AsciiDoc is co-equal first-class authoring format with YAML; markdown discouraged for technical artifacts | A8 + L633 |
| C5 | YAML is canonical machine-readable ground truth | A10 |
| C6 | Multi-format entry (md/yaml/adoc) — system normalizes to YAML | A10-e |
| C7 | Composition over unification for schemas — DO NOT build a unified registry+requirements metamodel | A12 |
| C8 | ELT/FW schemas are the base, not target — semantically strengthen, do not redesign from scratch | A12 |
| C9 | Migration path mandatory for every breaking change; deprecate-don't-delete; parallel-operation period | A8 |
| C10 | Design for IDE/dashboard decoupling — features must work standalone, not require full BMAD adoption (strategic threat: "if I continue with BMAD, it will likely be a custom implementation") | A8 |
| C11 | FSM substrate work = research; **no binary success criteria**; ceremonial-prose simplification is itself an open research question | A7 |
| C12 | `lkml-gen` is deprecated; all CLI capability via `lkml` | A4 correction |
| C13 | Verified-fixes ship before unverified-substrate (Evolutions before FSM) | L633 critique |
| C14 | mtime anchor for project-customization detection = `_bmad-output/` directory creation epoch (`stat %W` birthtime; ctime fallback) — NOT newest-artifact mtime | User correction 2026-04-29 |
| C15 | BMAD-METHOD repo is substrate, not consumer — exclude from "BMAD-using project" analysis | User clarification |
| C16 | Worktree dedup — per project, only the primary working tree is inspected; `.git/worktrees/` skipped | User scope rule |
| C17 | Schema-validated YAML registry as source of truth; step files are pure renderers; markdown `stepsCompleted` and XML `<step n>` ordinals are removed | A3 |
| C18 | The CSV-for-spreadsheet-review is the "big win" — LinkML model exposes spreadsheet column names mapped to YAML schema while preserving adoc fidelity | A12 |
| C19 | Approval discipline: propose change → justify → wait for user approval → then execute. **NO unprompted self-flagging cascades.** | Project CLAUDE.md scope-discipline; L670→L692 trace |
| C20 | Identifier schema (subsystem-prefixed hierarchical) is NOT deferrable — Wave 0 prerequisite per A11/A12 | A11 + A12 |

---

## §7 What Claude Got Wrong (anti-patterns; do NOT repeat in ultra-plan)

**A1.** **AsciiDoc characterized as "legacy"** → AsciiDoc is the user's PREFERRED authoring format for serious technical work. Markdown is the inadequate format. "Legacy" applies only to bash extraction scripts (`extract-section.sh` + awk parsers), not to adoc.

**A2.** **Wave 3 (FSM) before Wave 2 (verified Evolutions)** → backwards. Verified-in-situ fixes ship first.

**A3.** **Wave 4 conflated three unrelated concerns** (liaison + review-extract-diff + adoc tooling) → they share a surface adjective ("tooling") but live at different substrate layers.

**A4.** **Unified LinkML metamodel covering registry + requirements** → A12 explicit reject. "The AND is signal that composition may be indicated. Why is one better than multiple? Are you mixing concerns?" Decoupled schemas with shared discipline.

**A5.** **Sham binary criteria** ("≤ 50% token threshold," "no behavioral regression," "drop prose markers if hypothesis holds") for FSM work → ceremonial prose carries overlapping load (state enforcement, behavioral steering, audit trail, fallback behavior, training-data alignment) that Claude cannot disentangle from outside the domain. **No pass/fail framing for open research questions.**

**A6.** **BMAD-METHOD repo included in "BMAD-using project" analysis** → substrate, not consumer.

**A7.** **mtime anchor on newest artifact** → wrong because output artifacts are written constantly during normal use. Use `_bmad-output/` directory creation epoch.

**A8.** **`lkml-gen` references** → deprecated; use `lkml`.

**A9.** **Numbering convention as "deferrable"** → A11/A12 reject; identifier schema is Wave 0 prerequisite.

**A10.** **Reactionary sycophant overcorrection** when called out — meta-caveat blocks, observer-only-research-mode, stripping risk register, "all decisions to user" abdication → loses analytical contribution. The middle path is **engaged engineering judgment with appropriate humility about training-distribution limits**, not abdication.

**A11.** **Unprompted self-flagging cascades** — when user critiques element X, address X. Do NOT preemptively revise adjacent elements Y/Z. If Y/Z appears suspect, surface as a question, not as an edit.

**A12.** **Bulk-reading large files into subagent prompts** — use iterative-retrieval pattern (dispatch with minimal context + search capability; evaluate; refine; resolve). Prevents 3,500-line bulk-read context exhaustion.

**A13.** **Faking BMAD workflow execution** — follow actual workflow steps; write artifacts to canonical paths (`_bmad-output/`, not ad-hoc); adopt facilitator stance per skill's MANDATORY EXECUTION RULES.

**A14.** **Treating discovery findings as "approximations"** when user has named exact files, exact line ranges, exact behaviors — reproduce them precisely; do not paraphrase.

**A15.** **"Stress test"-style brainstorm divergence with 100+ ideas** when the deliverable is a structured ranked menu — match technique cardinality to outcome shape; breadth-per-decision over volume-of-ideas.

---

## §8 Open Questions (ultra-plan should NOT close these)

| # | Question | Source |
|---|---|---|
| OQ1 | Which CALM/C4 metamodel variant to integrate (CALM vs C4 vs hybrid; existing CALM metamodel work referenced needs to be located) | A12 |
| OQ2 | Branch strategy — stay on `enhance/better-adr-capture` for upstream OR fork branch for custom-implementation per A8 strategic threat | A8 + M25 |
| OQ3 | Whether YAML-as-exclusively-authoritative is feasible in user's pragmatic case | A8 |
| OQ4 | Whether `bmad-liaison` should be promoted now or deferred — pattern observed in 2 projects, may not generalize | Q8/A8 |
| OQ5 | Naming/path of governance file: `governance.yaml` vs `project-governance.yaml` vs `_bmad/governance.yaml` | A11 |
| OQ6 | Choice of pilot workflow for M18 FSM experiment (recommend `bmad-create-architecture`; user may prefer different) | A7 |
| OQ7 | Choice of pilot agent for M19 (recommend `bmad-master`; user may prefer different) | A7 |
| OQ8 | Whether Q12 expansion-pack pattern (QA, BAML) is in-scope for this branch or follow-on | A12 |
| OQ9 | Whether spreadsheet column-name mapping for M21 is one schema-wide spec or per-artifact-type | A12 |
| OQ10 | Whether install-version-awareness (M24) needs schema flagging or is purely advisory | BG-2 |

---

## §9 Files inventory (for ultra-plan to reference)

**Stable artifacts on this branch (do not regenerate; reference):**
- `fixes/metaprompt-discover-and-integrate.md` (29 KB) — v2 with mtime-anchor correction
- `fixes/discovery/00-prior-context.md` (36 KB) — P-0..P-2 + verdicts + FR/RS deficiencies + linkml prior-analysis + answer-driven shifts
- `fixes/discovery/01-project-inventory.md` (14 KB) — BG-2 full report
- `fixes/discovery/03-master-catalogue.md` (11 KB) — 65 findings across 6 tiers, dependency DAG
- `fixes/discovery/04-evolution-audit.md` (30 KB) — BG-1 full audit + per-evolution punch list
- `fixes/discovery/05-open-questions.md` (16 KB) — Q3-Q12 with user inline answers (A3-A12 verbatim)
- `fixes/evolution-{1..4}/` — historical evidence; reference, do not patch
- `~/repos/linkml/_bmad-output/learnings/draft-bmad-improvement-analysis.md` (1434 lines)
- `~/repos/linkml/_bmad-output/learnings/bmad-upstream-contribution-strategy.md` (632 lines)

**Superseded / rejected (do not consume):**
- `fixes/discovery/04-integration-plan.md` — old plan, REJECTED by user
- `fixes/discovery/04-integration-plan-overreach.md` — sycophant overcorrection captured for reference only
- `fixes/discovery/brainstorming-calm-discovery-next-moves-2026-04-30.md` — brainstorm session redirected mid-Phase-1; procedural artifact only

**This brief:** `fixes/discovery/06-ultra-plan-handoff.md` — authoritative handoff document.

---

*End of brief. Hand to `gsd-ultraplan-phase`.*
