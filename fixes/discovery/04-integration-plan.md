# 04 — Integration Plan (Phase 6)

**Date:** 2026-04-29
**Branch:** `enhance/better-adr-capture` (after rebase onto `main` per A5)
**Sources:** `00-prior-context.md`, `03-master-catalogue.md`, user answers Q3-Q12 in `05-open-questions.md`.

---

## Sequencing principle (REVISED — user critique 2026-04-29)

**Three corrections to the prior draft:**
1. **Verified fixes before unverified substrate.** Evolutions 1-4 are already validated in user working directories (linkml's commit `d5dbbdb0a` etc.). They ship FIRST after Wave 1. The FSM substrate is research/aspirational and must build ON the proven fixes, not the inverse.
2. **AsciiDoc is a Wave 1 first-class authoring format**, NOT a Wave 4 legacy concern. Markdown is the inadequate format; AsciiDoc is preferred for serious technical work (per A8). The "legacy" label applies only to the bash/awk extraction scripts, never to adoc itself.
3. **Decompose conflated tooling concerns.** Liaison (cross-project coordination), review-extract-diff (substrate primitive), and adoc (authoring format) are three separate things. They were bundled because they all looked "tooling-shaped." They aren't.

**Revised sequence:**

1. **Wave 0 (per A11) — Project-level governance scaffolding.** Substrate-agnostic. Declares format policy including AsciiDoc-first option.
2. **Wave 1 — Substrate format & schema.** Decoupled schemas (per A12). **AsciiDoc + YAML are co-equal first-class authoring formats; markdown supported but discouraged for technical artifacts.** Multi-format entry pipeline + review-extract-diff loop are intrinsic to substrate (not separate wave).
3. **Wave 2 (was Wave 3) — Apply verified Evolutions 1-4** against Wave 1 substrate. Proven fixes for proven problems. The fix files in `fixes/evolution-{1..4}/` are retargeted (per A6) to current `src/bmm-skills/` paths but carry the same validated intent. No FSM dependency.
4. **Wave 3 (was Wave 2) — Real FSM substrate (research/pilot, per A7).** Builds on Wave 2-stabilized substrate. Pilot one workflow + one agent. Tests the token-budget hypothesis. May simplify (not block) Evolution-1 markers as a follow-on cleanup.
5. **Wave 4 — Decomposed tooling promotion.** Each item independently scopable; some may defer. (Liaison alone, possibly.)
6. **Wave 5 — Hygiene.** Dedup, install-version awareness, gitignore patterns.

Each wave: single coherent theme, atomic commits per slice, smoke-validation defined, single-revert rollback, `bmad-builder:bmad-workflow-builder` for new workflow scaffolding.

---

## Wave 0 — Project-level Governance Scaffolding

**Goal:** before any substrate work ships, projects must declare their authoritative sources, identifier schema, and curation discipline. Required regardless of BMAD/GSD/pro-workflow per A11.

**Deliverables:**
- New skill `bmad-init-governance` (uses `bmad-builder:bmad-workflow-builder`):
  - Generates `governance.yaml` at project root with: `identifier_schema`, `authoritative_sources`, `format_policy`, `pav_discipline`, `curation_locations`.
  - `identifier_schema` is enterprise-wide-unique + hierarchical per A12 (e.g., `{enterprise}-{project}-{type}-{nn}`, with `{enterprise}` namespace declared at init).
  - `authoritative_sources` enumerates ground-truth artifact locations (e.g., `requirements/*.adoc` OR `requirements/*.yaml`, with one designated canonical).
  - `format_policy` documents accepted entry formats (md/yaml/adoc per A10) and the YAML-as-ground-truth policy.
- New skill `bmad-validate-governance` — checks that each artifact in `authoritative_sources` has a PAV record, that identifiers match the declared schema, and that no orphan artifacts exist.

**Wave 0 commits (sequential):**
1. `feat(skill): add bmad-init-governance with governance.yaml schema`
2. `feat(skill): add bmad-validate-governance for orphan/identifier checks`
3. `docs(governance): document PAV discipline and identifier-schema conventions`

**Validation gate:** instantiate `bmad-init-governance` against `linkml/`, `architecture/`, `archbox/` in dry-run mode; verify the generated `governance.yaml` round-trips with their existing identifier patterns (FWA, ELT, LKML).

**Out-of-scope:** does NOT modify any existing skill. Adds new skills only.

---

## Wave 1 — Substrate Format & Schema (decoupled, per A12)

**Goal:** address the 28 substrate findings (F010-F028) from the catalogue. **Composition over unification** per A12. Each artifact type gets its own LinkML schema; binding is at the meta-level via PAV + governance.

**Stage 1a — Registry schema redesign (E3 substrate):**
- New schema: `bmm-workflow-status.linkml.yaml` (LinkML model). Fields:
  - `name` becomes `id` with required prefix (e.g., `wf-{project}-{name}`)
  - Split overloaded `status` → separate fields: `expected` (enum), `state` (enum), `output` (path or list)
  - Add `phase`, `description`, `last_updated` (existing)
  - Add `rationale` (per FR-3 / RS-6), `provenance` (PAV per A10)
  - Multi-output support: `output` as list-valued (per RS-1, kai/Elements pattern)
  - Versioning: `schema_version` at top + `output_version` per entry
- Migration script (Python, F1 promotion target): converts existing `bmm-workflow-status.yaml` instances (4 in the wild) to new schema with backward-compatible read.
- Updated template in `~/.claude/config/bmad/templates/bmm-workflow-status.template.yaml` (note: outside this repo, document-only here).

**Stage 1b — Requirements/UC/US/SS/DM/AC schemas (FR substrate):**
- ELT/FW schemas as starting point per A12 (closest to ideal but semantically weak).
- Per-type LinkML schemas: `requirement.linkml.yaml`, `use-case.linkml.yaml`, `user-story.linkml.yaml`, `subsystem.linkml.yaml`, `data-model.linkml.yaml`, `acceptance-criterion.linkml.yaml`.
- All schemas reference the project-level `governance.yaml` for identifier validation.
- Each schema includes: `id` (governed format), `title`, `rationale` (REQUIRED — Q9), `traces_to[]` (PAV), `version`, `phase`, `strength`, `source`, `test_procedure[]`.
- Cross-reference mechanics (per FR-4): `traces_to` entries are typed (UC→US, US→FR, FR→test, etc.) — same shape across schemas.

**Stage 1c — ADR schema (Q9 headline deliverable for `enhance/better-adr-capture`):**
- New schema: `architecture-decision.linkml.yaml`. Fields: `id`, `status` enum, `context`, `decision`, `consequences[]`, `rationale` (REQUIRED, the headline fix), `alternatives[]`, `links[]`, plus PAV.
- Update `src/bmm-skills/3-solutioning/bmad-create-architecture/architecture-decision-template.md` to render from this schema.
- Adoc-rendered ADR template included for projects that prefer adoc-authoritative.

**Stage 1d — AsciiDoc as first-class authoring format + multi-format entry pipeline (per A10 + A8):**
- AsciiDoc is the **preferred** authoring format for serious technical work. YAML is co-equal as the canonical machine-readable form. Markdown is supported for backwards compatibility but **discouraged** for technical artifacts.
- New skill `bmad-normalize-source` — accepts adoc/yaml/md as input (priority order); emits canonical YAML conforming to the appropriate Wave-1 schema; preserves PAV provenance back to source.
- Bidirectional renderer (per A10 inverted flow): YAML → adoc (preferred) or md (compatibility) for review and downstream consumption.
- Import path for projects with authoritative adocs: a Python rewrite of `extract-section.sh` and the four awk parsers, integrated as `bmad-import-from-adoc`. The bash/awk scripts themselves remain available, referenced as the predecessor implementation, but the canonical import path is Python.
- AsciiDoc-specific support: the `[#{idprefix}<id>]` anchor mechanic, `<<file.adoc#id,display [id]>>` cross-reference shape, and `.adoc-index` files (architecture/elt/) are all promoted as substrate primitives.

**Stage 1e — Review-extract-diff loop as substrate primitive (per A10 e + Q10):**
- New skill `bmad-diff-yaml` — produces a canonical, deterministic diff between proposed YAML and baseline YAML. Stable key ordering, no comment churn.
- New skill `bmad-review-change-proposal` — wraps the loop end-to-end: take edited source (any of adoc/yaml/md), normalize via `bmad-normalize-source`, diff via `bmad-diff-yaml`, present to operator, accept/reject/iterate.
- This is the **default** mechanism for "C → continue" transitions in any workflow whose output has a derivable YAML form (per Q10/A10 e — agile process, multi-format-entry, normalize, derivative for review).

**Wave 1 commits (logical groups, atomic per stage):**
1. `feat(schema): add bmm-workflow-status linkml model and migration script`
2. `feat(schema): add requirement/UC/US/SS/DM/AC linkml models referencing governance`
3. `feat(adr): canonical ADR schema with required rationale + alternatives + PAV (Q9 headline)`
4. `feat(skill): bmad-normalize-source for adoc/yaml/md entry`
5. `feat(skill): bmad-render-derivative (YAML → adoc preferred / md compatibility)`
6. `feat(skill): bmad-import-from-adoc — Python rewrite of extract-section + parse-{uc,us,ss,dm}`
7. `feat(skill): bmad-diff-yaml deterministic canonical diff`
8. `feat(skill): bmad-review-change-proposal end-to-end review-extract-diff loop`

**Validation gates per stage:**
- 1a: migrate luka, teague, linkml, kai/Elements registries to new schema; verify `bmad-validate-governance` accepts all four.
- 1b: round-trip ELT/FW extracted YAMLs through new schemas; verify no field loss against the original adoc.
- 1c: convert linkml's `_bmad-output/learnings/another-bmad-ADR-screwup-wtf.txt` (ADR capture failure case) into the new schema; verify rationale is captured.
- 1d: pipe an architecture/elt adoc through normalize → derive → diff; expect zero-diff round trip on unchanged input.

**Out-of-scope per A12:** unified metamodel covering all artifact types. Schemas are decoupled.
**Out-of-scope per A8:** absorbing `lkml` toolchain or its templates. Reference only.

---

## Wave 2 — Apply verified Evolutions 1-4 (per critique 2026-04-29)

**Goal:** ship the proven prose fixes against the new Wave 1 substrate. These have been **validated in working directories** (linkml's commit `d5dbbdb0a` for E2; user's E4 application to `~/.claude/commands/bmad/dev-story.md`; E1 markers proven in linkml/architecture). They solve real problems with prose that has been seen to work. They ship **before** any FSM experimentation.

**Why before Wave 3:** putting verified fixes after the unproven FSM substrate would gate known-good work on speculative work. The FSM substrate may simplify these fixes later (Wave 3 cleanup), but it must not block them.

**Stage 2a — Evolution-1 install (per E1 punch list):**
- Apply E1 markers (FACILITATOR, A/P/C, NEVER-generate-without-input, FORBIDDEN-to-load-next-step, stepsCompleted) to:
  - `bmad-correct-course/checklist.md` and `workflow.md`
  - `bmad-check-implementation-readiness/steps/step-02-prd-analysis.md` and `step-06-final-assessment.md` (install from `fixes/evolution-1/`)
- Expand to: `bmad-validate-prd/steps-v/*`, `bmad-create-epics-and-stories/steps/*`, `bmad-generate-project-context/steps/*`, `bmad-create-story/workflow.md`, `bmad-quick-dev/workflow.md`.

**Stage 2b — Evolution-2 install (per E2 punch list):**
- Install `step-01b-continue.md` + `State Recovery Check` workflow.md section in the 5 original P0/P1/P2 targets (`bmad-create-epics-and-stories`, `bmad-check-implementation-readiness`, `bmad-generate-project-context`, `bmad-sprint-planning`, `bmad-create-story`).
- For the 4 skills that already have `step-01b-continue.md` via different effort, add the explicit `State Recovery Check` section so the gate is authoritative (currently relies on implicit step-01-init routing).
- The `step-01b-continue-*.md` content from `fixes/evolution-2/` is the patch material, retargeted to current `src/bmm-skills/` paths.

**Stage 2c — Evolution-3 install (per E3 punch list):**
- Each completion step gets the registry-writeback patch from `fixes/evolution-3/workflow-completion-writeback-patch.md`, retargeted to write against the new Wave-1 registry schema.
- Replace generic "update workflow status (if exists)" prose with E4-form: "MANDATORY. If exists update; if not exists create from template" — this eliminates the `do X (if Y)` defect class corpus-wide.
- Template additions (per E3 D3): `create-epics-and-stories` row in `bmm-workflow-status.template.yaml` — note: outside this repo, document-only here.

**Stage 2d — Evolution-4 install (per E4 punch list):**
- `bmad-dev-story/workflow.md` Step 1: change `<action if="story file inaccessible">HALT</action>` to `<action>CREATE artifact at canonical path using prior-story template</action>`.
- Step 10: add MANDATORY artifact-create-or-update directive (mirror the slash-command patch already applied outside this repo).
- `bmad-create-story/workflow.md`: same treatment.
- Audit ALL completion steps for `do X (if Y)` patterns; convert to MANDATORY-with-create-fallback.

**Stage 2e — Cross-cut: Party Mode persistence (linkml Cluster 4):**
- New skill `bmad-deliberation-persist`: every A/P/C menu's deliberation outcome (advanced elicitation result, party-mode decision) writes to a YAML decision log immediately, before any "C → continue."
- Closes F067 (Party Mode decisions not persisted) and the Evolution-1 C4 intent.

**Wave 2 commits:**
1. `feat(e1): install evolution-1 in correct-course and check-implementation-readiness`
2. `feat(e1): expand evolution-1 to validate-prd, epics-and-stories, project-context, create-story, quick-dev`
3. `feat(e2): install evolution-2 state recovery in 5 original P0/P1/P2 targets`
4. `feat(e2): authoritative State Recovery Check section in 4 already-partial skills`
5. `feat(e3): registry writeback patch in completion steps; eliminate "if exists" defect class`
6. `feat(e4): bmad-dev-story Step 1 creates artifact; Step 10 mandatory; audit corpus-wide`
7. `feat(skill): bmad-deliberation-persist for party-mode + advanced-elicitation outcomes`

**Validation gates:**
- BG-1 audit re-run shows all punch-list items closed.
- Sandbox dev-story with no pre-existing artifact: artifact created at canonical path with all required sections.
- Sandbox party-mode run: every decision logged to YAML before "continue."
- linkml registry round-trips through new Wave-1 schema and the writeback machinery.

**Out-of-scope:** any FSM substrate work. Pure prose-fix application, ridden onto Wave-1's new schemas/formats.

---

## Wave 3 — Real FSM Substrate (research/pilot, per A7)

**Goal:** replace prose-FSM with executable FSM as a substrate **upgrade** to the already-stabilized Wave-2 prose fixes. LLM as operator/facilitator. Per A7: stage 3a (workflow step files) first, then 3b (pilot agent activation).

**Why after Wave 2:** Wave 2 ships verified-in-situ fixes against the new Wave-1 substrate. Wave 3 is a substrate-upgrade hypothesis test. The prose discipline must work first; the FSM is then an opportunity to simplify (not a precondition).

**Stage 3a — Workflow FSM substrate:**
- New module: a deterministic state-machine runner (Python preferred per kickoff; TS/JS option for IDE integration).
  - States, transitions, gate predicates expressed as YAML conforming to a new schema `workflow-fsm.linkml.yaml` (composed with Wave-1 schemas, not unified per A12).
  - Verifiable gate tests (assertions evaluated against project state, not LLM judgment) per A7.
  - Workflow step files become **renderers against current FSM state**, not control flow.
- Convert `bmad-create-architecture` first (highest user-exercise + already-Wave-2-stable). Compare token budget pre/post.
  - **Hypothesis test (per A7):** "verbose ceremonial preambles are heritage from less capable LLMs." Measure: post-FSM step file ≤ 50% pre-FSM token count, with no behavioral regression. If hypothesis holds, the "LOAD the FULL workflow.md" preamble is removed (FSM enforces order; LLM does not need to be shouted at). If it fails, FSM stays as observer-only and prose remains authoritative.
- **E1 marker simplification (Wave-2 cleanup):** in FSM substrate, "FORBIDDEN to load next step until C is selected" becomes an FSM gate predicate, not prose discipline. The prose marker is dropped from converted step files. FACILITATOR and NEVER-generate-without-input semantics stay (these are LLM-discipline rules, not state-machine concerns).
- **E2 simplification:** state recovery is an FSM property — reconstructible from project state at any time. The `step-01b-continue.md` files installed in Wave 2 become **redundant** for FSM-converted skills; their installation persists for non-FSM skills until corpus rollout completes.
- **E3 simplification:** registry writeback becomes an FSM observer (every state transition emits a registry update event). The Wave-2 patch-based writeback continues to work; the FSM observer is an upgrade path that eliminates per-step prose.

**Stage 3a per-skill rollout (priority order):**
1. `bmad-create-architecture` — pilot (already Wave-2 stable; high user exercise).
2. `bmad-create-epics-and-stories` — second pilot (closes the highest-impact substrate dependency: state-recovery + writeback as FSM properties rather than prose).
3. Remaining skills on rolling rollout, only after the two pilots prove the hypothesis.

**Stage 3b — Agent activation FSM (pilot per A7(c)):**
- Convert `bmad-master.md`'s `<activation critical="MANDATORY">` block into a real FSM definition. Greeting / config-load / menu-display become states with executable transitions.
- Compare token budget and behavioral fidelity pre/post.
- If pilot succeeds, schedule corpus rollout in a follow-on wave (out of scope here).

**Wave 3 commits:**
1. `feat(fsm): workflow-fsm linkml schema + Python runner`
2. `refactor(skill): bmad-create-architecture on FSM substrate; drop redundant prose markers`
3. `refactor(skill): bmad-create-epics-and-stories on FSM substrate; writeback as observer`
4. `feat(fsm): pilot bmad-master agent activation as FSM (Stage 3b)`
5. `test(fsm): token-budget hypothesis report (pre-vs-post per converted skill)`

**Validation gates:**
- Per skill: end-to-end run against a sandbox project produces artifact output identical to (or improved over) the Wave-2 prose-fixed version.
- Token budget: post-FSM step file ≤ 50% pre-Wave-2 token count. **Stop rollout if any skill regresses.**
- Recovery: kill mid-workflow; resume; expect clean restoration without manual `stepsCompleted` editing.

**Out-of-scope:** corpus-wide agent activation conversion (pilot only). Removing Wave-2 prose patches before FSM hypothesis is proven.

---

## Wave 4 — Tooling Promotion (deprecate, do not delete)

**Goal:** absorb the architecture-repo extraction infrastructure as referenced/deprecated tooling. Per A8: cannot abandon scripts; deprecate them.

**Stage 4a — Extraction-script reference:**
- Reference (don't copy) `architecture/elt/extract-section.sh` and `architecture/tools/parsers/parse-{uc,us,ss,dm}.awk` in BMAD documentation as **legacy import tools** for projects with adoc-authoritative sources.
- New BMAD skill `bmad-import-legacy-adoc`: wraps invocation of these scripts (or a Python re-implementation per kickoff hint), produces Wave-1-canonical YAML.
- Per A8: usable for "case of authoritative adocs"; deprecated in favor of multi-format entry pipeline (Wave 1d).

**Stage 4b — `bmad-liaison` skill (F083 promotion):**
- Cross-project coordination primitive present in linkml + archbox.
- New skill: declares mailbox protocol (inbox/outbox, message schema with PAV), governance.yaml integration.
- A4-category — closes the worktree-isolation gap.

**Stage 4c — `bmad-review-extract-diff` skill (Q10 → A10 e):**
- Codifies the review-edit-extract-diff loop as a BMAD primitive.
- Inputs: source artifact path (md/yaml/adoc), baseline YAML.
- Outputs: derived YAML diff for operator review.
- Selective default for high-value workflows (PRD, architecture, story creation, dev-story per Q10 recommendation).

**Wave 4 commits:**
1. `feat(skill): bmad-import-legacy-adoc wrapping extract-section.sh + parsers as deprecated path`
2. `feat(skill): bmad-liaison cross-project coordination primitive`
3. `feat(skill): bmad-review-extract-diff for change-proposal loop`

**Validation gates:**
- Import skill: round-trip an architecture/elt adoc → YAML → adoc with zero loss against the original.
- Liaison skill: simulate cross-project message exchange between two sandbox projects.
- Diff skill: feed a small adoc edit through the loop; expect a readable YAML diff.

---

## Wave 5 — Hygiene

**Goal:** address E1-E4 hygiene findings (F100-F103).

**Stage 5a — Install dedup (F100):**
- Document the canonical install path; note that vendored `_bmad/` should not be edited (use new project-level governance.yaml + skill overrides).
- Optional installer flag for symlink-vs-copy install mode.

**Stage 5b — Install-version awareness (F101):**
- `bmad-init-governance` records the BMAD version in `governance.yaml`.
- Skills check version compatibility on invocation.

**Stage 5c — Config precedence + gitignore patterns (F102, F103):**
- Document config precedence: `governance.yaml` > `bmad/config.yaml` > `_bmad/bmm/config.yaml`.
- Adopt linkml's governance VCS strategy (Cluster 3 pattern): `.gitignore` includes peripheral output, excludes governance.

**Wave 5 commits:**
1. `docs(install): canonical install path and dedup recommendation`
2. `feat(install): version awareness on skill invocation`
3. `docs(install): config precedence and gitignore patterns`

---

## Rebase + integration mechanics (per A5)

**Pre-Wave-0:**
1. `git fetch origin main`
2. `git rebase origin/main` (the 4 enhance commits replay onto current main HEAD)
3. Resolve any conflicts in `fixes/` (low surface, mostly additive).
4. Note: the evolution patches in `fixes/evolution-{1..4}/` reference old `_bmad/bmm/workflows/...` paths. They are **historical evidence**, not patch-ready content (per A6). Wave 3 retargets them onto the new `src/bmm-skills/` shape.

**Per-wave commits land directly on `enhance/better-adr-capture` after rebase.**

**Pre-merge to main:**
- All wave validation gates pass.
- `compound-engineering:document-review` parallel persona pass on `04-integration-plan.md`.
- `pro-workflow:plan-interrogate` walk of the decision tree.
- One sandbox project (recommend `linkml/`) runs the full Wave 0+1+2 stack end-to-end.

**Out-of-scope (declare):**
- Modifying user-project working trees during integration.
- Promoting any local fix without canonical BMAD restatement.
- Slash-command surface in `~/.claude/commands/bmad/` (separate workstream).
- Replacing markdown wholesale with AsciiDoc (per A8: enable AsciiDoc + YAML; do not eliminate markdown).
- Custom-implementation fork (per A8: user may go custom; integration plan must keep features decoupled).

---

## Risk register

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| FSM substrate (Wave 3) is too ambitious for one branch | Medium | High | Pilot one workflow first; if token-budget hypothesis fails, fall back to prose with FSM as observer-only |
| LinkML adoption stalls Wave 1 | Medium | High | Use plain YAML+JSON-Schema as fallback; LinkML is preferred but not load-bearing |
| User goes custom-implementation (per A8) | Medium | Medium | All Wave 1+ deliverables work standalone; document decoupling clearly |
| `_bmad/` vendor-install dedup breaks projects | Low | High | Wave 5 only; long deprecation period; opt-in symlink mode |
| Schema drift between independently-evolved per-type schemas (per A12 composition) | Medium | Medium | Shared discipline via PAV + governance.yaml; periodic cross-schema review |
| Evolution patches in `fixes/evolution-{1..4}/` referenced as truth post-merge | Low | Medium | Add `STATUS: HISTORICAL EVIDENCE` headers; link to Wave commit ranges |

---

## Acceptance for "complete the evolution fixes" mission

Per kickoff: "you will complete the evolution fixes, rebase with main (merge as necessary), AND identify the improvements need in BMAD to characterize the benefits of structured asciidoc + YAML + scripts (perhaps rewritten in python) over vanilla markdown. I also want isolate the silly state machine hiding inside BMAD workflows sickness."

| Mission item | Wave coverage |
|---|---|
| Complete the evolution fixes | Wave 3 closes E1-E4 punch list against new Wave 1+2 substrate |
| Rebase with main | Pre-Wave-0 mechanics |
| Identify improvements for adoc + YAML + scripts (Python) | Wave 1 (substrate redesign) + Wave 4 (script promotion as deprecated) |
| Isolate state-machine sickness | Wave 2 (real FSM substrate; pilot then expand) |

**This plan exits when** all five waves' validation gates pass on a sandbox project end-to-end and `compound-engineering:document-review` clears the plan for merge to main.

---

## Open issues NOT addressed by this plan

Surfaced for user decision (live in `05-open-questions.md` follow-ups):

- IDE/dashboard integration (per A8) — strategic threat to BMAD adoption itself; out of scope here.
- CALM/C4 Architecture metamodel integration (per A12) — captured as future Wave aspiration, not Wave 1-5 deliverable.
- BAML / QA / other lkml expansion-pack integration (per A12) — same.
- Decision on YAML-as-exclusively-authoritative (per A8 "I have not decided whether the YAML as exclusively authoritative is yet feasible") — Wave 1 supports both modes; final policy is a project-level choice via governance.yaml.
