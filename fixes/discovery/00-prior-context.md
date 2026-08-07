# 00 — Prior Context (Pre-Flight P-0 + P-1 + P-2)

**Session:** 2026-04-29
**Branch:** `enhance/better-adr-capture`

---

## User pointers (P-0 — verbatim)

### Q1 — State-machine sickness, where?
> Q1 a — Anything with multiple steps and state progression
> c — is the current state (template maybe)

**Interpretation (confirmed in chat):** the FSM is everywhere multi-step workflows progress (any workflow with `<step n="N">` or "next step" semantics is a hidden FSM). `bmm-workflow-status.yaml` is the *correct* substrate for that state but should be **template-driven** (state-as-data) rather than ad-hoc YAML mutated by each workflow's final step.

### Q2 — S3 proximal evidence in `~/repos/architecture/`
> the existence or processing of .adoc files, and episodic references to lost rationale in the ADR when adocs were mined for PRD requirements. It was quite a production, the files were too large to process naively, so the scripts to process adocs were created
> /home/randyg/repos/architecture/elt/extract-section.sh

**Concrete evidence node:** `/home/randyg/repos/architecture/elt/extract-section.sh`
**Failure mode:** ADR rationale lost when adocs were mined for PRD requirements.
**Token-budget signal:** files too large to process naively → utility-script-driven extraction was the workaround.

### Q3 — Template direction (registry as schema-driven YAML vs declarative FSM vs both)
> Q3 not sure, some versions of bmad used yaml to track state, others used markdown.

**Material data, not deferral.** BMAD's own substrate has oscillated between YAML and markdown for state tracking. The "right" direction is not a clean a-priori choice — it depends on what Phase 3 audits actually surface across the project corpus. Action: in Phase 3, every per-project file records `state_tracking_format` ∈ {yaml, markdown, mixed, none} so Phase 5 synthesis can argue from evidence rather than preference. Q3 stays in `05-open-questions.md` as a synthesis-time decision but with empirical inputs.

---

## Substrate facts (P-3, P-4 partial)

**Repo restructured** since the evolution docs were written: `src/modules/bmm/` no longer exists. Current shape:
- `src/bmm-skills/{1-analysis, 2-plan-workflows, 3-solutioning, 4-implementation}/` — phase directories of native skill packages
- `src/core-skills/` — bmad-* shared skills (advanced-elicitation, brainstorming, party-mode, etc.)

Each skill package: `SKILL.md`, `bmad-skill-manifest.yaml`, `workflow.md`, optional `steps/`, optional `templates/`, `data/`. This is a substantially different surface than the `_bmad/bmm/workflows/...` paths the evolution docs reference. **All four evolutions must be re-targeted for this surface.**

**Evolution-1 verdict — MATERIALLY APPLIED for deliberative authoring skills, ABSENT for the validation skills the fix files were actually written for.** (BG audit, see `04-evolution-audit.md`)
- Applied (full 5-marker pattern): `bmad-create-architecture`, `bmad-create-ux-design`, `bmad-create-prd` (steps-c), `bmad-create-product-brief`. Research workflows use a deliberate `[C]-only` variant (4 markers, no A/P/C menu — by design).
- **Absent: `bmad-correct-course/`** (zero FACILITATOR/A/P/C markers) and **`bmad-check-implementation-readiness/`** (zero markers across all 6 step files) — these are precisely the two skills `fixes/evolution-1/checklist.md` and `step-02-prd-analysis.md` / `step-06-final-assessment.md` were authored for. Highest-leverage upstreaming target for E1.

**Evolution-2 verdict — 0 OF 5 ORIGINAL TARGETS APPLIED; 4 OTHER SKILLS PARTIAL VIA DIFFERENT MECHANISM.** (BG audit)
- 4 step-01b-continue.md files exist with correct STATE-RECOVERY semantics in `bmad-create-product-brief`, `bmad-create-prd`, `bmad-create-ux-design`, `bmad-create-architecture` — but reached only via implicit step-01-init routing.
- **NO workflow.md anywhere contains the explicit `State Recovery Check` section** called for in the spec. The gate is not authoritative even where step-01b exists.
- All 5 original P0/P1/P2 targets (`bmad-create-epics-and-stories`, `bmad-check-implementation-readiness`, `bmad-generate-project-context`, `bmad-sprint-planning`, `bmad-create-story`) carry NO step-01b file.
- Highest-leverage target: E2-P0 = epics-and-stories (workflow.md gate + step-01b-continue.md + step-01 Section-0 guard).

**Evolution-3 verdict — SPEC UNIMPLEMENTED in src/, BUT registry instances exist in the wild with substantial schema drift.** (BG audit + user-supplied registry instances 2026-04-29)
- The string `bmm-workflow-status.yaml` does not appear anywhere in `src/`.
- 4 completion steps carry generic "update workflow status (if exists)" prose — no registry path resolution, no template auto-create, no fallback chain.
- Cross-cuts E4: every "if exists" in completion steps is the same `do X (if Y)` defect class.

**In-the-wild registry instances inspected:**

| project | path | format | drift evidence |
|---|---|---|---|
| luka | `docs/bmm-workflow-status.yaml` | `workflow_status:` list | 9 entries, no `create-epics-and-stories`, status mixes file-path with `optional`/`skipped` |
| teague | `docs/bmm-workflow-status.yaml` | `workflow_status:` list | 11 entries with extra `command:` and `notes:` fields, project_name says "genmap" but path is "teague" (project rename drift) |
| linkml | `docs/bmm-workflow-status.yaml` | `workflow_status:` list | 9 entries INCLUDING `create-epics-and-stories`, header says "retroactive — state reconstructed from existing artifacts" (confirms ad-hoc creation per Evolution-3 README) |
| kai/Elements | `Plans/Elements/bmm-workflow-status.yaml` | **flat top-level keys** (`prd:`, `architecture:` …) — entirely different schema | `prd:` is **list-valued** (5 PRDs across capability areas, with inline FR counts and status comments). The other three projects can't represent this. |

**Substrate deficiencies in the registry schema (D-tier evidence):**

| # | Deficiency | Observed in |
|---|---|---|
| RS-1 | Schema cannot represent multi-output workflows (e.g., multi-PRD projects) — kai/Elements broke the schema by switching to list-valued `prd:` and flat keys | kai/Elements vs others |
| RS-2 | `status` field is overloaded — values are either {required, recommended, optional, conditional, skipped} OR a file path OR ad-hoc descriptive prose. Should be split into `expected`, `state`, `output`. | All four |
| RS-3 | No `create-epics-and-stories` entry in luka, teague — confirms Evolution-3 D3 finding (template missing the row) | luka, teague |
| RS-4 | Field-set drift: `command:`/`notes:` (teague), `last_workflow`/`last_workflow_date`/`brainstorming:` (kai), `description:` length varying widely | All four |
| RS-5 | No subsystem prefix on `name` (e.g., `prd` not `prd-luka`) — same class as FR-2 in FR-extraction deficiencies | All four |
| RS-6 | No rationale, no traces_to, no version metadata at workflow-entry level — same class as FR-3, FR-8 | All four |
| RS-7 | No diff-friendly canonical form (varying key order, optional fields, comments interleaved with data) — blocks the review-edit-extract-diff loop | All four |
| RS-8 | Project rename drift not handled (teague's `project_name: "genmap"`) | teague |

**Conclusion: the registry schema is the same class of substrate problem as the requirements YAML schema.** Both should be governed by a shared metamodel (LinkML candidate). This collapses E3 from "fix the writeback" to "redesign the registry as governed schema, then fix the writeback to it" — a Wave 1 substrate change rather than a Wave 2 bolt-on.

---

## Linkml project's PRIOR ANALYSIS (BG-2 finding — must integrate, not duplicate)

The linkml project has already produced two substantial documents that overlap this very effort:

- **`/home/randyg/repos/linkml/_bmad-output/learnings/draft-bmad-improvement-analysis.md`** (1434 lines, 27 sections) — detailed deficiency analysis, contribution-readiness assessment, prioritized next actions.
- **`/home/randyg/repos/linkml/_bmad-output/learnings/bmad-upstream-contribution-strategy.md`** (632 lines, 12 sections) — categorization framework, priority improvements with scoring, execution plan, risk assessment.

**Their G/I/C scoring (Generality / Impact / Cost-of-skipping, each 0-5, max 15) for upstream contribution:**

| Cluster | Score | Maps to (my taxonomy) |
|---|---|---|
| **Cluster 0 — Workflow Resumption Metadata** ⭐ (frontmatter + resumption section + session naming) | 15/15 | A1 (Evolution-2) |
| **Cluster 3 — BMAD Governance VCS Strategy** ⭐ | 15/15 | E1 / E4 |
| **Cluster 4 — Party Mode Memory Loss Anti-Pattern** ⭐ | 15/15 | C4 + B1 (Evolution-1 + Evolution-4) |
| Cluster 1 — PRD Context Budget Management | 14/15 | D4 (token bloat) |
| Cluster 2 — Architecture Prerequisite Enforcement | 13/15 | A1 / A2 (Evolution-2) |
| Cluster 6 — Agent Customization Patterns | 12/15 | D4 + Q7 (FSM scope) |
| Cluster 5 — Progressive Help System | TBD | F3 |
| Cluster 7 — Domain-Specific Workflow Steps | TBD | C category |

**Contribution-readiness assessment (linkml's % confidence for upstreaming):**

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

**Action items per linkml's plan that overlap this effort:**
1. Action 1 (Validate Fork vs Upstream Divergence) — partially done by BG-1 audit; remaining work is the BMAD-fork vs BMAD-METHOD-canonical reconciliation.
2. Action 3 (Abstract Top 4 Improvements) — directly maps to my Wave 3 work; their abstractions are the starting point.

**Implication: do not re-derive a parallel priority taxonomy.** Adopt linkml's G/I/C scoring rubric and contribution-readiness percentages. Their Cluster 0 (Workflow Resumption Metadata) is essentially Evolution-2 generalized, with concrete frontmatter design and session-naming guidance already worked out. Wave 3 should consume that work rather than restart.

---

## Project inventory (BG-2 summary; full report in `01-project-inventory.md`)

**All 9 BMAD projects are pristine** when measured by the **correct anchor** — `_bmad-output/` directory's **creation epoch** (birthtime; ctime fallback), NOT the newest artifact mtime inside it. (User correction 2026-04-29: the latter is wrong because output artifacts are written constantly during normal use.) **Customization lives outside `_bmad/` proper** — in peripheral output dirs (`liaison/`, `failings/`, `learnings/`) and in promoted Python packages (now via the `lkml` tool, not the deprecated `lkml-gen`).

| Project | Customization intensity | Substrate-tier evidence |
|---|---|---|
| **linkml** | High | Evolution-2 already applied via tracked commit `d5dbbdb0a`; carries `liaison/`, `failings/`, `learnings/` peripheral dirs; the two BMAD analysis docs above; promoted Jinja2/Mustache adoc templates to `lkml-gen/src/lkml_gen/generators/templates/` |
| **architecture** | High | Owns the AsciiDoc→YAML toolchain: `tools/parsers/parse-{uc,us,ss,dm}.awk` (4 awk scripts, 524 lines total), `elt/extract-section.sh`, `.adoc-index` files, `elt/extracted/*.yaml`, `fw/extracted/*.yaml`. F1/D1/D3 promotion candidates. |
| **archbox** | Medium | `liaison/` mailbox protocol identical to linkml's (cross-project coordination primitive — A4-category candidate for new BMAD pattern); custom `_bmad-output/architecture/` doc layout |
| Factweave | Low | Content-only — no toolchain customization |
| padimae, describe, claudefat, teague, relloq | None | Pristine |

**Two BMAD install versions in the wild:** architecture + Factweave use older `bmad-create-product-brief`/`bmad-brainstorming` workflow naming; archbox + linkml use newer unprefixed naming. Both carry `step-01b-continue.md`. Confirms the substrate has been moving — Wave 0 may need to handle install-version awareness.

**Zero ad-hoc workarounds** (`WORKAROUND`/`HACK`/`TODO(BMAD)`/`FIXME`/`HOTFIX`) anywhere in inspected `_bmad-output/` trees. Cleanliness is a positive surprise.

**`liaison/` mailbox protocol** appears in BOTH linkml and archbox — distinct cross-project coordination primitive. Not in any taxonomy yet; A4 (worktree isolation / coordination) is closest fit. Candidate for new BMAD skill `bmad-liaison` if promoted.

**User clarification on workaround surface (2026-04-29):** *"The workarounds were the adhoc extraction of YAML from asciidoc with the scripts."*

The reason BG-2 found zero `WORKAROUND`/`HACK`/`FIXME` markers in `_bmad-output/` is that the actual workaround surface lives **outside** `_bmad-output/`: the extraction infrastructure itself (`extract-section.sh`, `tools/parsers/parse-{uc,us,ss,dm}.awk`, the now-deprecated `lkml-gen` Jinja2/Mustache templates) is the workaround. It works around BMAD's inability to consume structured authoritative source material. **Promotion candidate: deprecate, do not delete.** (Per A4 + A8.)

---

## User answers to Q3-Q12 (2026-04-29) — direction shifts

Captured verbatim in `05-open-questions.md`. Material design shifts driven by the answers:

### Wave 0 emerges (per A11)
**"Must begin with project-level initialization and GOVERNANCE of a LinkML-compatible document … with authoritative sources clearly specified, and meticulously curated, none of this slop dropping files wherever, and then languishing. … Required regardless of whether it is BMAD, GSD, or pro-workflow."**

A new Wave 0 prerequisite to all substrate work: project-level governance scaffolding that specifies identifier schema, authoritative sources, curation discipline. Cross-cuts BMAD/GSD/pro-workflow — meaning the deliverable is **substrate-agnostic** (not just a BMAD skill).

### Flow inverts (per A4(e) + A10(e))
**Original direction (and the architecture-repo workaround pipeline):** adoc → extract YAML → derived for downstream consumption.
**Revised direction:** **multi-format-entry (markdown OR YAML OR adoc) → normalize to YAML → YAML as ground truth → derive adoc/markdown for review**. Extraction scripts become legacy import tools, deprecated but available for case of authoritative adocs.

Per A10: *"YAML is simply easier to validate and version manage at the business object level, where as formatted texts is simply harder to mechanistically govern and align."* Operator edits proposed YAML; system generates adoc/markdown for review; differences adjudicated.

### PAV discipline as normative (per A10)
*"The use of PAV to document the relationship should be normative."* PAV (provenance / assertion / verification — confirmed by linkml memory entry "PAV-first traceability" commit `6c552e9`) is a traceability discipline that links derived artifacts back to authoritative source. To be **normative** in BMAD means every derived artifact carries PAV metadata.

### Real FSM, not prose-FSM (per A7)
*"LLMs are very poor state machines (i.e. a non-deterministic state machine == gambling)"*. Wave 2 must replace the ceremonial XML/markdown FSM with a real, deterministic, executable state machine. LLM is operator/facilitator, not state-keeper. The "LOAD the FULL workflow.md" preamble disappears once the FSM is real. Per A7 sequencing: **(a) workflow step files first, then (c) pilot agent activation** — incremental, evidence-driven.

User caveat: *"the verbose ceremonial preambles are heritage from less capable LLMs"* — simplification testing is itself a research question. Does the workflow still work without the preamble shouting once a real FSM enforces order? Wave 2 must include this as a measurable test.

### Composition, NOT unified metamodel (per A12)
*"Do not get too locked into a unified LinkML metamodel covering registry+requirements. The issue is broader than requirements. The AND is signal that composition may be indicated. Why is one better than multiple? Are you mixing concerns?"*

Q12 (a) is OFF the table. Recommended path: **decoupled schemas with shared meta-discipline.** Each artifact type (registry, requirements, UC/US/SS/DM/AC) has its own LinkML schema. They're bound by:
- **PAV traceability** between them (per A10)
- **Project-level governance doc** (per A11) specifying identifier schema (preferred: enterprise-wide unique + hierarchical) and authoritative sources
- **Optional CALM/C4 metamodel integration** for lifecycle traceability across CONOPS / requirements / user stories / use cases / architecture (per A12)
- **Expansion-pack pattern** — LinkML schemas for QA artifacts, BAML prompt-engineering, etc. (per A12)

The "big win" per user: *"a LinkML model that exposed the spreadsheet column names mapped to the YAML schema, while preserving the fidelity of the asciidoc source"* — i.e., the schema is a **bridge between three audiences**: spreadsheet-reviewers (human), YAML (machine), adoc (engineering rigor).

### ELT/FW schemas as base, not target (per A12)
*"The schema used in elt and fw are probably the closest to ideal, but the CSV generated for review was useful."* The current `extracted/*.yaml` schema is a starting point with known gaps (missing rationale, weak identifier semantics). Wave 1 enhances it; doesn't restart from scratch.

### Tooling correction (per A4)
**`lkml-gen` is no longer a tool.** All CLI capabilities now via `lkml` tool. Update all references in `04-integration-plan.md`.

### IDE/dashboard integration is the strategic threat (per A8)
*"the integration with IDEs and dashboards are terrible, no doubt exacerbated by the limitations exposed in this study. If I continue with BMAD, it will likely be a custom implementation."*

The integration plan must therefore design for **decoupling**: features should work standalone, without requiring full BMAD adoption. Substrate work that locks downstream consumers into BMAD-specific shape is high-risk.

### Migration discipline (per A8)
*"We need a migration path. We can't abandon the extraction scripts, but they can be deprecated."* Every breaking change in this work must ship with: (a) deprecation notice on the old path, (b) migration script (Python preferred), (c) parallel operation period.

### Reject Q12(a), Reject Q10(b/c) defaults
- Q12 answer rejects unified metamodel — pursue decoupled with shared discipline (effectively a hybrid of (b) and (d)).
- Q10 answer (option e) goes BEYOND (c) — agile multi-format-entry pipeline, not just diff-on-continue.

**Evolution-4 verdict — DEFECT PERSISTS IN SKILL WORKFLOW.** (BG audit)
- Slash-command was patched outside this repo (`~/.claude/commands/bmad/dev-story.md`).
- Canonical skill workflow `src/bmm-skills/4-implementation/bmad-dev-story/workflow.md` STILL exhibits the structural defect: Step 1 HALTs on missing story file rather than creating it; Step 10 only updates internal sections of an assumed-existing file. The slash-command patch did not cascade into the skill.

---

## Mission scope (per user kickoff message, 2026-04-29)

1. Complete Evolutions 1-3 (verify applied state, upstream what's missing).
2. Rebase `enhance/better-adr-capture` with `main` (merge as necessary).
3. Identify improvements needed in BMAD to characterize benefits of structured **AsciiDoc + YAML + scripts** (perhaps rewritten in Python) over vanilla markdown.
4. Isolate the **state-machine anti-pattern** hiding inside BMAD workflows.

---

## P-1 — Episodic memory hits (signature-grouped)

> Run only queries derived from P-0 user pointers, plus signature queries the user did not narrow.
> S1+S2 are largely closed by Evolutions 1/2/4 — verify, do not over-search.
> S3 has the strongest narrowing signal: architecture repo + adoc + extract-section.sh.

### Filesystem evidence captured (architecture repo)

- `/home/randyg/repos/architecture/elt/extract-section.sh` — bash script that mines a single `[#{idprefix}<id>]` section from large adoc files. **Failure mode it works around:** files too large to process naively (conops.adoc = 96 KB, data-models.adoc = 41 KB).
- `/home/randyg/repos/architecture/elt/*.adoc` — 5 large adocs (conops, data-models, subsystems, use-cases, user-stories) + `.adoc-index` files.
- `/home/randyg/repos/architecture/elt/extracted/*.yaml` — 4 extracted YAMLs (data-models 355 lines, subsystems 38, use-cases 1669, user-stories 152).
- `/home/randyg/repos/architecture/fw/` — sister directory: many SMALL adocs, one per artifact, organized in subsystem subdirs (`fusion/`, `bin/`, `lens/`, `pipeline/`, `workspace/`, `knarrative/`, `harvester/`, `knote/`, `manual/`, `smartmap/`). ID pattern: `{TYPE}-FWA-{NN}-{slug}.adoc` where TYPE ∈ {UC, US, SS}.
- `/home/randyg/repos/architecture/fw/extracted/{fwa-subsystems, fwa-user-stories, fwa-use-cases}.yaml` — same extraction pattern, smaller per-file because source adocs are smaller.

**Pattern observation (likely an evolution).** elt = first-attempt monolithic adocs needing extract-section.sh. fw = refactored one-file-per-artifact, no extraction script needed because each file IS already a section. Direction: **structured AsciiDoc → YAML, scripts as glue, schema emerging from project usage.**

**YAML schema observed in fw/extracted/fwa-user-stories.yaml:**
```
- id: "US-FWA-12"
  title: "..."
  acceptance_criteria:
    - version: "current"
      text: "..."
```
AC are **versioned** (`version: "current"`) — superior to vanilla markdown PRD prose.

**Lost-rationale failure surface:** the YAML schema captures `id`, `title`, `acceptance_criteria` but NOT `rationale` / `context` / `consequences`. When adocs are mined for PRD/requirements, the rationale lives in the adoc body but doesn't survive extraction. Direct evidence for D2 (ADR rationale weakness).

---

### S1 — Token usage / template bloat (CORPUS-WIDE EVIDENCE)

**Smoking gun: "LOAD the FULL workflow.md, READ its entire contents and follow its directions exactly!" preamble** appears across 8 projects in episodic memory:

| project | date | workflow | session jsonl |
|---|---|---|---|
| claudefat | 2026-03-05 | create-product-brief | `9c305089-3fbd-4caa-a35c-313be48e16c8.jsonl` (lines 48-174) |
| describe | 2026-03-07 | core/tasks (steps CRITICAL) | `9d6de955...jsonl` (lines 13-162) |
| describe | 2026-03-08 | core/tasks (steps CRITICAL) | `9d6de955...jsonl` (lines 1583-1724) |
| linkml-lkml--bmad | 2026-03-03 | quick-dev | `69825487...jsonl` (lines 159-185) |
| linkml-lkml--bmad | 2026-03-04 | quick-dev | `9376e055...jsonl` (lines 339-364) |
| linkml-lkml--bmad | 2026-03-05 | core/tasks (steps CRITICAL) | `9376e055...jsonl` (lines 3427-3470) |
| archbox-docs | 2026-03-06 | core/tasks (steps CRITICAL) | `a02d2cb6...jsonl` (lines 6256-6427) |
| archbox | 2026-04-03 | bmad-quick-flow/quick-dev | `7bfa2600...jsonl` (lines 1273-1281) |
| linkml | 2026-03-21 | bmad-quick-flow/quick-dev | `65c2ded7...jsonl` (lines 9937-9949) |
| linkml | 2026-04-17 | core/tasks (steps CRITICAL) | `e435b0ff...jsonl` (lines 72-91) |
| padimae | 2026-02-24 | 3-solutioning/create-architecture | `15cb41c7...jsonl` (lines 263-338) |

**Two preamble variants** (both patterns are FSM-encoded-in-prose):
1. **Workflow-level**: `IT IS CRITICAL THAT YOU FOLLOW THIS COMMAND: LOAD the FULL {workflow.md}, READ its entire contents and follow its directions exactly!`
2. **Task-level**: `IT IS CRITICAL THAT YOU FOLLOW THESE STEPS - while staying in character ... <steps CRITICAL="TRUE"> 1. Always LOAD the FULL {core/tasks}...`

The emphatic-prose-with-CRITICAL-banner is the smoking gun: BMAD knows the LLM doesn't reliably execute procedural markdown, so it shouts. The shouting is itself token bloat AND evidence the substrate is wrong.

### S2 — Workflow sloppiness / discipline drift

- **linkml 2026-04-01** [`25266018...jsonl` lines 332-344]: "I want claude to STOP doing BMAD incompetantly. I want BMAD to do BMAD, and I want it to work." — frustration session, motivating context for evolutions.
- **linkml 2026-03-31** [`753c25cd...jsonl` lines 457-1167]: direct path reference to `BMAD-METHOD.git/fixes/evolution-2` — Evolution-2 origin session.
- **linkml 2026-03-31** [`c3f38d79...jsonl` lines 184-186]: "see the problem, what is your solution?" — Evolution-2 design discussion.
- **linkml-lkml--bmad 2026-03-05** [`9376e055...jsonl` lines 3362-3406]: "This looks like you are IGNORING the BMAD plan, that is not acceptable." — sloppiness symptom.
- **Factweave 2026-03-24** [`6e9e64fc...jsonl`]: "no, WHO ARE YOU? claude or bmad agent?" — agent-identity drift, B-class symptom.

### S3 — Markdown over-reliance / AsciiDoc + YAML / state-machine substrate

- **Factweave 2026-03-25** [`64e2f818...jsonl` lines 345-346]: "Typically, you assign a system or component ID, such as FWA, ELT to the system or component. I can see this is going to cause friction in BMAD, so I am going to propose that we encode the requirements..." — **direct origin of the FWA/ELT ID convention; explicit collision with BMAD's requirements format.**
- **linkml 2026-03-27** [`86df1617...jsonl` lines 95-134]: "I don't want to diverge unnecessarily from the schema that was emerging from the factweave and element (elt) requirements exposition" — **schema-emerging-from-adoc-extraction is treated as authoritative; the linkml project consumes it.**
- **rfp-triage 2026-04-15** [`ea3ebfb5...jsonl` lines 208-211]: "Archbox architecture has been described prior to reimplementation, it helped to resolve confusion over the objective" — positive validation: structured architecture-as-adoc paid off downstream.
- **linkml 2026-04-21** [`4cbfea2e...jsonl` lines 372-463]: "1-5. Approved." — recent ADR-rationale approval discussion (need to read for content).
- **padimae 2026-02-21** [`919279c0...jsonl` lines 2715-2725]: "DOT language ... DSL binding is bespoke" — possible ADR rationale context.

### Cross-cutting

- **State machine sickness ≡ "LOAD the FULL" preamble pattern.** The FSM is the markdown file; every tick re-loads it; the LLM is shouted at because procedural prose is unreliable substrate.
- **bmm-workflow-status.yaml** appeared in only 1 hit (archbox 2026-04-03) — under-represented in conversation, suggests it's not yet load-bearing in practice (consistent with Evolution-3's "spec only, patches pending" status).
- **`extract-section.sh` script pattern + `extracted/*.yaml`** is the user's already-implemented workaround for D1/D2/D3/D4 — direct evidence of the desired substrate direction. Script is bash; user hinted "perhaps rewritten in python."

---

## FR extraction — deficiencies exposed (user pointer, 2026-04-29)

The adoc → YAML extraction process for elements / FactWeave was not just a mechanical conversion — it was a **diagnostic** that exposed multiple BMAD substrate deficiencies. Per user:

> "BMAD numbering was not adequate from the best practices POV. In fact, most of that extraction process exposed NUMEROUS deficiencies that must be addressed to avoid loss of fidelity."

Cross-references the Factweave 2026-03-24 episodic hit: BMAD's default `FR-001` (restart-at-001-per-document) is a **"ROOKIE move in serious system engineering."** ENCOSE-approved practice requires:

- **Subsystem-prefixed IDs** — `FR-FWA-001`, not bare `FR-001`. Stable across documents. No conflation across subsystems.
- **Requirement traceability from cradle to grave** — through test procedures back to provenance.
- **Capability-area organization** as primary unit, not flat enumeration.
- **Versioned acceptance criteria** (e.g., `version: "current"` vs `version: "future"`) — confirmed in `fwa-user-stories.yaml`.
- **No lossy transformation** — the prose adoc carries semantic relationships (cross-references, "Includes VM-US-001, VM-US-040, VM-US-043, …") that flat YAML must preserve.

**Specific deficiencies exposed by FR extraction (taxonomy mapping):**

| # | Deficiency | Taxonomy |
|---|---|---|
| FR-1 | BMAD restarts requirement numbering at -001 per document | B4 (numbering drift) |
| FR-2 | No subsystem identifier in requirement IDs | B4 / D3 |
| FR-3 | Schema lost `rationale`, `source`, `traces_to`, `phase`, `strength`, `test_procedure` during implementation despite being in the proposed schema | D2 / D3 |
| FR-4 | Cross-reference mechanics (`<<file.adoc#{idprefix}ID,display [ID]>>`) have no equivalent in markdown-only PRDs | D1 / D5 |
| FR-5 | "Includes" provenance lines (`Includes VM-US-001, VM-US-040, …`) lost in YAML extraction | D2 |
| FR-6 | No traceability matrix from FR → UC → US → DM → test procedure | D3 |
| FR-7 | Capability area / subsystem hierarchy implicit in adoc structure but not explicit in the YAML metamodel | D5 |
| FR-8 | Versioning of acceptance criteria implemented per-AC; no version metadata at the artifact level | D3 |

These are NOT downstream sloppiness — they are substrate gaps. Wave 1 D-tier work must address them as **first-class requirements of the canonical schema**, not bolt-on later.

---

## Review-edit-extract-diff change-proposal loop (substrate pattern)

User pointer (2026-04-29):

> "the adoc review + edit + YAML extraction + diff is a very good change proposal loop that's worth noting. This matches similar agent/operator collaboration cycles."

**The loop:**
1. **Operator edits** the human-readable adoc (authoring surface).
2. **Agent extracts** YAML from the edited adoc (machine-readable surface).
3. **System diffs** the new extracted YAML against the prior committed YAML.
4. **Operator reviews** the diff as the change-proposal artifact — same role as a PR diff in code review.
5. **Operator accepts/rejects/refines** — accepted diffs commit; rejected diffs trigger another adoc edit cycle.

**Why this matters as a BMAD pattern:**
- It puts the **operator in the authoring seat** and the **agent in the extraction seat** — exactly the Evolution-1 "FACILITATOR not content generator" intent, expressed as a workflow loop.
- The **diff** is the canonical communication artifact between operator and agent — far better than narrative summaries, more honest than agent self-reports.
- It generalizes to **any agent/operator collaboration cycle**: operator edits source, agent extracts/transforms, diff against baseline becomes the review surface. Examples: PRD → epic YAML → diff; ADR → decision YAML → diff; story.adoc → story-status.yaml → diff.
- It is **invariant under substrate change** — works equally well for adoc, markdown, code, or any human-edited source whose structured form is mechanically derivable.

**Implications for Wave 1/2/3:**
- Wave 1 (substrate): the canonical extracted-YAML schema must be diff-friendly (deterministic key ordering, stable IDs, no implicit defaults that vary by extractor version).
- Wave 2 (state machinery): workflow state transitions can themselves be expressed as YAML diffs — `bmm-workflow-status.yaml` becomes a diff log.
- Wave 3 (artifact discipline): every multi-step workflow's "C → continue" transition produces a diff for operator review before commit. Replaces or augments the A/P/C menu's current opaque "save" action.

This pattern is **operationally proven** in the architecture repo and is a candidate for canonical promotion alongside the parsers/templates (Q8 in `05-open-questions.md`).

---

## P-2 — Session journal extracts (only files named by P-1 hits)

### 64e2f818 — Factweave, 2026-03-24 (requirements-as-YAML origin)

User's original framing of the substrate:
> "encode the requirements in yaml, corresponding to the information model represented in adocs, then we can use a jinja template to generate markdown format if that is what you needed... adoc is far superior to markdown for serious engineering, but it should be mechanically generated, same with markdown for requirements... proper structuring, all these things are tractable with simple schemes, templates and generators... Eventually, we'll define a **LinkML Model for the requirements**, which DEFINE what the schema properties MEAN."

User's proposed YAML schema (per session, then partially implemented):
```yaml
FR-FWA-KA-001:
  system: FWA
  subsystem: KA
  requirement: "..."
  source: PRD1+FRI
  phase: MVP
  strength: Strong
  rationale: "..."         # ← rationale was IN THE PROPOSED SCHEMA
  traces_to:
    - journey: Elena-J1
    - principle: P2-facts-are-claims
  test_procedure: TP-FWA-KA-001
```

**Implemented schema in `fw/extracted/fwa-user-stories.yaml`** has only `id`, `title`, `acceptance_criteria` — `rationale`, `source`, `traces_to`, `phase`, `strength`, `test_procedure` are all MISSING. **Confirmed mechanism for D2 (ADR rationale weakness): the schema lost rationale during implementation, so adoc → YAML extraction throws it away.**

ID convention established here: `{TYPE}-{SYSTEM}-{NN}` — TYPE ∈ {SS, UC, US, DM, FR, AC}, SYSTEM ∈ {FWA, ELT, …}.

### 86df1617 — linkml, 2026-03-27 (schema reuse + parser/generator infrastructure)

User pinned the schema scope:
> "I don't want to diverge unnecessarily from the schema that was emerging from the factweave and element (elt) requirements exposition... FWA and ELT both had a very well defined structure that laid out nicely in asciidoc. This is not that, or even close."
> "...Each adoc TYPE embodies an implicit schema — the metamodel underneath."
> "If you read the first 300 lines of each type, then write an awk parser, we can probably convert them mechanically to yaml."

**Existing infrastructure in user projects (promotion candidates):**
- **`~/repos/architecture/tools/parsers/`** — adoc-type-aware parsers that extract YAML from adoc per-type (UC, US, SS, DM, AC, ConOps).
- **`lkml-gen/src/lkml_gen/generators/templates/`** — 12 Jinja2 + Mustache adoc generator templates for the same 6 types.
- **`_bmad-output/planning-artifacts/extracted/`** (in linkml project) — extracted YAML data conforming to the shared schema.

This is the round-trip pipeline: adoc ↔ YAML via parsers (one direction) and Jinja/Mustache templates (the other), with LinkML schema as the eventual governance layer. **None of this is in BMAD-METHOD upstream.**

### 25266018 — linkml, 2026-04-01 (BMad Master's self-diagnosis)

After the "fuck me" session and saved feedback rule `feedback_bmad_execution_discipline.md`, the bmad-master agent itself articulated the deepest problem:

> **"BMAD is instructions. Claude is the engine. There is no separate BMAD runtime that can 'assert control' — when you invoke a BMAD workflow, Claude reads the workflow.xml and instructions, and it's Claude's discipline (or lack of it) that determines whether the protocol is followed."**

> **"The framework has real gaps. Claude has real discipline problems with workflow execution. Both need fixing."**

The bmad-master agent file's own activation block is direct evidence of the FSM-in-prose anti-pattern (`<activation critical="MANDATORY">` with `<step n="1..9">` ordinals). This is XML-encoded FSM expressed as prompt instructions — not a state machine running anywhere, just imperatives the LLM is exhorted to self-execute. The shouting (`🚨 IMMEDIATE ACTION REQUIRED`, `MANDATORY`, `CRITICAL="TRUE"`) is the substrate's own admission that procedural prose is unreliable.

**Direct quote from saved feedback rule (`feedback_bmad_execution_discipline.md`):**
> "STOP bypassing BMAD workflows... Claude pretending to do BMAD" is the anti-pattern. Either execute the workflow or don't invoke it.

This rule alone is a stable artifact in the user's memory, but it patches the symptom (Claude's discipline) rather than the cause (the prose-as-FSM substrate).
