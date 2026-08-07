# Evolution Audit — Canonical `src/` Tree

**Date:** 2026-04-29
**Branch:** `enhance/better-adr-capture`
**Scope:** Verify which parts of Evolutions 1-4 have actually been applied to
the canonical `src/` tree at `/home/randyg/repos/harness/skills/BMAD-METHOD.git/`.

## Method

For each evolution, scan the canonical skills tree
(`src/bmm-skills/{1-analysis,2-plan-workflows,3-solutioning,4-implementation}/`)
for the textual markers introduced by that evolution's patch and compare to the
target list named in `fixes/evolution-N/`. The original evolution docs reference
older `_bmad/bmm/workflows/...` paths; those have been remapped to native skill
packages of the form `bmad-<workflow-name>/{SKILL.md,workflow.md,steps/}`.

Markers searched:

| Evolution | Markers |
|-----------|---------|
| E1 (deliberative discipline) | `FACILITATOR`, `A/P/C`, `FORBIDDEN to load next step`, `stepsCompleted`, `NEVER generate content` |
| E2 (state recovery) | `step-01b-continue.md` file existence, `State Recovery Check` section in `workflow.md` |
| E3 (writeback) | `bmm-workflow-status.yaml`, `workflow_status_file`, `Workflow Status Registry` block |
| E4 (mandatory artifact) | `MANDATORY — always required`, `Create or update story document`, `if exists` escape-hatch removal |

---

## Section 1 — Evolution-1 (Deliberative Discipline)

The Evolution-1 source files in `fixes/evolution-1/` actually span THREE
workflows (not just architecture):

1. `step-02-context.md` → `bmad-create-architecture`
2. `step-03-core-experience.md` → `bmad-create-ux-design`
3. `step-04-architectural-patterns.md` → `bmad-technical-research`
4. `step-02-prd-analysis.md` and `step-06-final-assessment.md` →
   `bmad-check-implementation-readiness`
5. `checklist.md` → `bmad-correct-course`

So the audit must score five skills, not one.

### 1.1 `bmad-create-architecture/steps/`

All six middle steps carry all 5 markers; init/continue/complete carry the
subset appropriate for their role (terminal steps don't need A/P/C menu).

| File | (a) FACILITATOR | (b) A/P/C menu | (c) FORBIDDEN to load next | (d) stepsCompleted | (e) NEVER generate content | Status |
|------|-----------------|----------------|----------------------------|---------------------|-----------------------------|--------|
| `step-01-init.md` | YES | — | — | YES | YES | applied (init scope) |
| `step-01b-continue.md` | YES | — | — | YES | YES | applied (continuation scope) |
| `step-02-context.md` | YES | YES | YES | YES | YES | **applied (full)** |
| `step-03-starter.md` | YES | YES | YES | YES | YES | **applied (full)** |
| `step-04-decisions.md` | YES | YES | YES | YES | YES | **applied (full)** |
| `step-05-patterns.md` | YES | YES | YES | YES | YES | **applied (full)** |
| `step-06-structure.md` | YES | YES | YES | YES | YES | **applied (full)** |
| `step-07-validation.md` | YES | YES | YES | YES | YES | **applied (full)** |
| `step-08-complete.md` | YES | — | — | YES | YES | applied (terminal scope) |

**Naming drift:** Evolution-1 source `step-03-core-experience.md` does NOT
correspond to architecture's `step-03-starter.md` — `core-experience` is the
UX-design skill (see 1.2). The architecture step-03 is `step-03-starter.md`
and is fully patched.

### 1.2 `bmad-create-ux-design/steps/`

| File | FACILITATOR | A/P/C | FORBIDDEN | stepsCompleted | NEVER generate | Status |
|------|-------------|-------|-----------|----------------|----------------|--------|
| `step-01-init.md` | YES | — | — | YES | YES | applied (init) |
| `step-01b-continue.md` | YES | — | — | YES | YES | applied (continuation) |
| `step-02-discovery.md` | YES | YES | YES | YES | YES | applied (full minus 5 markers, 10 hits) |
| `step-03-core-experience.md` | YES | YES | YES | YES | YES | **applied (full)** |
| `step-04-emotional-response.md` | YES | YES | YES | YES | YES | **applied (full)** |
| `step-05-inspiration.md` | YES | YES | YES | YES | YES | **applied (full)** |
| `step-06-design-system.md` | YES | YES | YES | YES | YES | **applied (full)** |
| `step-07-defining-experience.md` | YES | YES | YES | YES | YES | **applied (full)** |
| `step-08-visual-foundation.md` | YES | YES | YES | YES | YES | **applied (full)** |
| `step-09-design-directions.md` | YES | YES | YES | YES | YES | **applied (full)** |
| `step-10-user-journeys.md` | YES | YES | YES | YES | YES | **applied (full)** |
| `step-11-component-strategy.md` | YES | YES | YES | YES | YES | **applied (full)** |
| `step-12-ux-patterns.md` | YES | YES | YES | YES | YES | **applied (full)** |
| `step-13-responsive-accessibility.md` | YES | YES | YES | YES | YES | **applied (full)** |
| `step-14-complete.md` | partial | — | — | YES | — | applied (terminal scope) |

### 1.3 `bmad-create-prd/steps-c/`

All discovery/elicitation steps carry the 5 markers (some fewer hits because
the UX-only `ABSOLUTELY NO TIME ESTIMATES` line is omitted in PRD).

| File | FACILITATOR | A/P/C | FORBIDDEN | stepsCompleted | NEVER generate | Status |
|------|-------------|-------|-----------|----------------|----------------|--------|
| `step-01-init.md` | YES | — | — | YES | YES | applied (init) |
| `step-01b-continue.md` | YES | — | — | YES | YES | applied (continuation) |
| `step-02-discovery.md` | YES | YES | YES | YES | YES | applied |
| `step-02b-vision.md` | YES | YES | YES | YES | YES | applied |
| `step-02c-executive-summary.md` | YES | YES | YES | YES | YES | applied |
| `step-03-success.md` … `step-10-nonfunctional.md` | YES | YES | YES | YES | YES | applied (all 8 elicitation steps) |
| `step-11-polish.md` | partial | YES | partial | YES | partial | applied (polish scope) |
| `step-12-complete.md` | partial | — | — | YES | — | applied (terminal scope) |

### 1.4 `bmad-create-product-brief/steps/`

| File | FACILITATOR | A/P/C | FORBIDDEN | stepsCompleted | NEVER generate | Status |
|------|-------------|-------|-----------|----------------|----------------|--------|
| `step-01-init.md` | YES | — | — | YES | YES | applied (init) |
| `step-01b-continue.md` | YES | — | — | YES | YES | applied (continuation) |
| `step-02-vision.md` | YES | YES | YES | YES | YES | **applied (full)** |
| `step-03-users.md` | YES | YES | YES | YES | YES | **applied (full)** |
| `step-04-metrics.md` | YES | YES | YES | YES | YES | **applied (full)** |
| `step-05-scope.md` | YES | YES | YES | YES | YES | **applied (full)** |
| `step-06-complete.md` | partial | — | — | YES | — | applied (terminal) |

### 1.5 `bmad-check-implementation-readiness/steps/`

The Evolution-1 fix files include `step-02-prd-analysis.md` and
`step-06-final-assessment.md` for THIS skill — but they intentionally do
NOT use A/P/C (auto-progressing assessment, not deliberative authoring).
Markers found: **NONE.** The skill's six step files contain 0 instances of
FACILITATOR/A/P/C/FORBIDDEN/stepsCompleted/NEVER generate.

| File | Status |
|------|--------|
| `step-01-document-discovery.md` | absent (no markers) — but workflow design doesn't require A/P/C |
| `step-02-prd-analysis.md` | partial — fix file in `fixes/evolution-1/` says "FACILITATOR" + "NEVER generate content" expected; current src has neither |
| `step-03-epic-coverage-validation.md` | absent |
| `step-04-ux-alignment.md` | absent |
| `step-05-epic-quality-review.md` | absent |
| `step-06-final-assessment.md` | partial — fix file expects FACILITATOR + NEVER generate content; current src has neither |

### 1.6 `bmad-correct-course/`

No `steps/` directory exists. Workflow-only structure
(`workflow.md` + `checklist.md` + `SKILL.md`). The Evolution-1
`checklist.md` was the patched form; current `src/` `checklist.md` is the
older mechanical form (no FACILITATOR posture, no A/P/C, no halt-conditions
on deliberative gates beyond what was always there).

| File | FACILITATOR | A/P/C | FORBIDDEN | stepsCompleted | NEVER generate | Status |
|------|-------------|-------|-----------|----------------|----------------|--------|
| `workflow.md` | — | — | — | — | — | **absent** |
| `checklist.md` | — | — | — | — | — | **absent** |
| `SKILL.md` | — | — | — | — | — | absent (delegates to workflow.md) |

### 1.7 Research workflows

Evolution-1 includes `step-04-architectural-patterns.md` for technical research.
That file DOES use a [C]-only menu (no A or P) by design, with FORBIDDEN gates
and stepsCompleted. The current `bmad-technical-research/technical-steps/` and
`bmad-domain-research/domain-steps/` and `bmad-market-research/steps/` carry
4 markers each (FORBIDDEN, stepsCompleted, NEVER generate, plus a
related text but **no FACILITATOR and no A/P/C**). This is `[C]-continue-only`
discipline — a deliberate variant for research (no menu choice; just a save
gate).

| Skill | Steps with [C]-continue discipline | Status |
|-------|-------------------------------------|--------|
| `bmad-technical-research/technical-steps/` | step-01..06 (all 6) | **applied (research-variant)** |
| `bmad-domain-research/domain-steps/` | step-01..06 (all 6) | **applied (research-variant)** |
| `bmad-market-research/steps/` | step-01..06 (all 6) | **applied (research-variant)** |
| `bmad-market-research/market-steps/` | step-01..06 (all 6) | applied (duplicate / legacy folder) |

### 1.8 Evolution-1 — missing-from-target-list

These multi-step skills produce deliberative content but do not currently
carry the Evolution-1 5-marker pattern, and they were not in the original
fix-file target list:

| Skill | Why it should adopt E1 pattern |
|-------|-------------------------------|
| `bmad-check-implementation-readiness` | Deliberative readiness gating — every checklist step is a judgement call. The Evolution-1 fix file `step-02-prd-analysis.md` was authored for THIS skill but never installed. |
| `bmad-correct-course` | Sprint Change Proposal is heavyweight deliberative work. The fix file `checklist.md` was authored for THIS skill but never installed. |
| `bmad-validate-prd/steps-v/*` | 13 validation steps, each a critical deliberative decision. Currently no FACILITATOR/A/P/C posture. |
| `bmad-create-epics-and-stories/steps/*` | 4 steps, each producing user-facing artifacts. Currently no FACILITATOR/A/P/C/stepsCompleted. |
| `bmad-generate-project-context/steps/*` | 3 steps with no deliberative gating. |
| `bmad-create-story/workflow.md` | Generates story spec — should be deliberative, not auto-pilot. |
| `bmad-quick-dev/step-*.md` | Quick path bypasses deliberation by design — explicitly opt-out, but worth confirming. |

---

## Section 2 — Evolution-2 (Workflow State Recovery Gate)

Evolution-2 expects two things in every long-running step-file workflow:

1. A `State Recovery Check` section in `workflow.md` that routes to
   `step-01b-continue.md` when output exists with `stepsCompleted` frontmatter.
2. A `step-01b-continue.md` step file that handles continuation.

### 2.1 Skills with `step-01b-continue.md` already installed (4)

| Skill | Path | Has `State Recovery Check` in `workflow.md`? | Status |
|-------|------|-----------------------------------------------|--------|
| `bmad-create-product-brief` | `src/bmm-skills/1-analysis/bmad-create-product-brief/steps/step-01b-continue.md` | NO | **partial** (step file present, workflow.md gate missing) |
| `bmad-create-prd` | `src/bmm-skills/2-plan-workflows/bmad-create-prd/steps-c/step-01b-continue.md` | NO | **partial** |
| `bmad-create-ux-design` | `src/bmm-skills/2-plan-workflows/bmad-create-ux-design/steps/step-01b-continue.md` | NO | **partial** |
| `bmad-create-architecture` | `src/bmm-skills/3-solutioning/bmad-create-architecture/steps/step-01b-continue.md` | NO | **partial** |

The 4 step-01b files DO contain proper STATE RECOVERY semantics: each begins
with "Workflow Continuation Handler", reads existing document frontmatter,
inspects `stepsCompleted`, and resumes at the appropriate step. **However**,
no `workflow.md` has the up-front `State Recovery Check` section — the gate
relies on `step-01-init.md` calling step-01b internally (see 1.1 entries
where `step-01-init.md` references step-01b-continue). This is a softer form
of the Evolution-2 gate than the spec calls for.

### 2.2 Original Evolution-2 P0/P1 targets — file present?

| Target Skill | Has `step-01b-continue.md`? | Has `State Recovery Check`? | Status |
|--------------|------------------------------|-----------------------------|--------|
| `bmad-create-epics-and-stories` (P0) | NO | NO | **absent** |
| `bmad-check-implementation-readiness` (P1) | NO | NO | **absent** |
| `bmad-generate-project-context` (P1) | NO | NO | **absent** |
| `bmad-sprint-planning` (P2) | N/A — no `steps/` dir; workflow-only skill | N/A | **absent** (different shape; needs `instructions.md`-style preamble in `workflow.md`) |
| `bmad-create-story` (P2) | N/A — no `steps/` dir; workflow-only skill | N/A | **absent** (different shape; needs `<step n="0">` recovery gate in `workflow.md`) |

### 2.3 Evolution-2 — missing-from-target-list

These step-file skills produce persistent output and would benefit from the
recovery gate but were not in the original target list:

| Skill | Output | Reason to include |
|-------|--------|-------------------|
| `bmad-validate-prd` (`steps-v/`) | validation report | 13 sequential validation steps; expensive to redo across sessions |
| `bmad-code-review` (`steps/`) | review report | 4 steps; long-running cross-file review |
| `bmad-domain-research/domain-steps/` | research doc | 6 deliberative steps |
| `bmad-technical-research/technical-steps/` | research doc | 6 deliberative steps |
| `bmad-market-research/steps/` | research doc | 6 deliberative steps |
| `bmad-quick-dev/step-*.md` | tech spec | 5 sequential steps |

---

## Section 3 — Evolution-3 (Workflow Status Registry Write-Back)

Evolution-3 expects every workflow's final step to write back to a central
`bmm-workflow-status.yaml` registry with explicit path resolution and
template-driven auto-create on missing file.

### 3.1 Per-file marker scan

Searched for: `bmm-workflow-status.yaml`, `workflow_status_file`,
`Workflow Status Registry`, explicit YAML registry write semantics.

| File | Generic "update workflow status" mention? | Registry path resolved? | Template auto-create? | Status |
|------|--------------------------------------------|-------------------------|------------------------|--------|
| `bmad-create-product-brief/steps/step-06-complete.md` | YES (lines 35,40,138,146) | NO | NO | **partial** (mentions only) |
| `bmad-create-prd/steps-c/step-12-complete.md` | YES (lines 10,12,18,25,50,52,55,93,101) | NO (uses `workflow_status["prd"]`) | NO | **partial** (placeholder syntax) |
| `bmad-create-ux-design/steps/step-14-complete.md` | YES (lines 10,12,18,27,74,76,78,79,97,104,135,157) | NO (uses `workflow_status["create-ux-design"]`) | NO | **partial** (placeholder syntax) |
| `bmad-create-architecture/steps/step-08-complete.md` | YES (lines 56, 63 — only in success/failure metrics) | NO | NO | **partial** (lip service only) |
| `bmad-create-epics-and-stories/steps/step-04-final-validation.md` | NO | NO | NO | **absent** |
| `bmad-check-implementation-readiness/steps/step-06-final-assessment.md` | NO | NO | NO | **absent** |
| `bmad-generate-project-context/steps/step-03-complete.md` | NO | NO | NO | **absent** |
| `bmad-sprint-planning/workflow.md` | NO | NO | NO | **absent** |
| `bmad-create-story/workflow.md` | NO | NO | NO | **absent** |
| `bmad-dev-story/workflow.md` | NO | NO | NO | **absent** (writes only to `sprint-status.yaml`, not central registry) |
| `bmad-domain-research/.../step-06-research-synthesis.md` | YES (line found in text scan) | NO | NO | **partial** (mentions, no spec) |
| `bmad-technical-research/.../step-06-research-synthesis.md` | YES | NO | NO | **partial** |
| `bmad-market-research/.../step-06-research-completion.md` | YES | NO | NO | **partial** |

### 3.2 Confirmation of "zero markers" finding

The original audit hint was "zero writeback markers anywhere in `src/`."
Refining: the **specific** registry filename `bmm-workflow-status.yaml` and
the explicit path-resolution / template-auto-create patterns from Evolution-3
appear in **zero** files in `src/`. What exists is generic "update workflow
status (if exists)" prose with no path, no fallback chain, no template
substitution, and no creation logic. So the finding stands: **Evolution-3 is
not applied — only the older lightweight precursor wording exists.**

### 3.3 Evolution-3 — missing-from-target-list

Every completion step in `src/bmm-skills/` should adopt the writeback patch.
The original target list named 5 (product-brief, prd, ux-design, architecture,
epics-and-stories). Add these:

| Skill | Final step | Reason |
|-------|------------|--------|
| `bmad-check-implementation-readiness` | `step-06-final-assessment.md` | produces readiness report — registry name `solutioning-gate-check` |
| `bmad-generate-project-context` | `step-03-complete.md` | produces `project-context.md` — should be tracked |
| `bmad-sprint-planning` | `workflow.md` (terminal block) | produces sprint plan |
| `bmad-validate-prd` | `steps-v/step-v-13-report-complete.md` | validation report |
| `bmad-domain-research` | `step-06-research-synthesis.md` | research doc |
| `bmad-technical-research` | `step-06-research-synthesis.md` | research doc |
| `bmad-market-research` | `step-06-research-completion.md` | research doc |
| `bmad-correct-course` | `workflow.md` (terminal block) | produces sprint change proposal |
| `bmad-retrospective` | `workflow.md` | produces retro report |
| `bmad-code-review` | `steps/step-04-present.md` | produces review report |

---

## Section 4 — Evolution-4 (dev-story Mandatory Story Artifact)

Evolution-4's patch targets **`~/.claude/commands/bmad/dev-story.md`** (the
slash-command), NOT a skill file. The commit `a2c4d268` only added files
under `fixes/evolution-4/` — confirmed. Now check whether the equivalent
discipline made it into `src/bmm-skills/4-implementation/bmad-dev-story/`.

### 4.1 Per-file scan

| File | `MANDATORY — always required` | `Create or update story document` | `if exists` escape-hatch | Required-sections enumeration | Status |
|------|--------------------------------|------------------------------------|---------------------------|--------------------------------|--------|
| `SKILL.md` | NO | NO | N/A | NO | absent (delegates to workflow.md) |
| `workflow.md` | NO | NO | YES (3 occurrences, all benign — `if exists` for sprint-status, project-context loaders, NOT for story-doc creation) | NO | **absent** |
| `checklist.md` | NO | NO | N/A | NO | absent |

The dev-story workflow's Step 9 + Step 10 only updates the story file's
internal sections (Status, File List, Dev Agent Record, Change Log) and
sprint-status.yaml. It nowhere instructs the agent to **create** the story
artifact file if missing. The story file is assumed to pre-exist (Step 1
discovers a `ready-for-dev` story by reading `sprint-status.yaml` and then
opens `{implementation_artifacts}/{story_key}.md`). If that file isn't
there, the workflow halts at "story file inaccessible" — there is no
fallback that creates the artifact.

This is the **structural** version of the same defect Evolution-4
addressed in the slash-command: the workflow assumes the artifact exists
and silently does nothing when it doesn't.

### 4.2 Evolution-4 — missing-from-target-list

| Skill | Defect type | Action |
|-------|-------------|--------|
| `bmad-dev-story` (skill) | Workflow-form of the same `if exists` defect — no MANDATORY artifact-creation gate when story file is missing | Apply the same patch wording to `workflow.md` Step 1 (`<action if="story file inaccessible">`) and Step 10 (artifact write-back) |
| `bmad-create-story` | Conversely, the story-creation skill should EMIT artifacts in the canonical artifact path; needs explicit path enumeration for new projects | Add MANDATORY-output specification |
| `bmad-quick-dev/step-oneshot.md` | One-shot path may also bypass artifact creation | Audit + add gate |
| Any "complete" step (E3 affected ones) | Same generalization: `do X (if Y)` where X is mandatory is a latent defect | Replace with `do X (MANDATORY). If Y: update. If not Y: create.` |

The principle from Evolution-4's "Generalization" section
(`do X (if Y)` is a latent defect when X is mandatory) applies broadly to:

- Every "update workflow status (if exists)" line in completion steps (E3 overlap)
- "Update story file (if exists)" patterns
- Any conditional save/write that an agent could rationalize away

---

## Section 5 — Consolidated Upstreaming Punch List

Ordered by dependency (E1 deliberative posture is foundation; E2 builds on
stepsCompleted; E3 builds on completion-step structure; E4 generalizes
mandatory-action wording).

### 5.1 Evolution-1 (deliberative discipline) — install patches

| Priority | File | Action |
|----------|------|--------|
| P1 | `src/bmm-skills/3-solutioning/bmad-check-implementation-readiness/steps/step-02-prd-analysis.md` | Install fix from `fixes/evolution-1/step-02-prd-analysis.md` (FACILITATOR + NEVER-generate gate) |
| P1 | `src/bmm-skills/3-solutioning/bmad-check-implementation-readiness/steps/step-06-final-assessment.md` | Install fix from `fixes/evolution-1/step-06-final-assessment.md` |
| P1 | `src/bmm-skills/4-implementation/bmad-correct-course/checklist.md` | Install fix from `fixes/evolution-1/checklist.md` (deliberative posture, halt conditions) |
| P2 | `src/bmm-skills/4-implementation/bmad-correct-course/workflow.md` | Add FACILITATOR / NEVER-generate-without-input preamble |
| P2 | `src/bmm-skills/2-plan-workflows/bmad-validate-prd/steps-v/step-v-*.md` (13 files) | Add FACILITATOR + A/P/C gating per E1 pattern |
| P2 | `src/bmm-skills/3-solutioning/bmad-create-epics-and-stories/steps/step-*.md` (4 files) | Add FACILITATOR + A/P/C gating |
| P3 | `src/bmm-skills/3-solutioning/bmad-generate-project-context/steps/step-*.md` (3 files) | Add FACILITATOR + minimal C-gating |
| P3 | `src/bmm-skills/4-implementation/bmad-create-story/workflow.md` | Add deliberative posture in story drafting block |

### 5.2 Evolution-2 (state recovery) — install patches

| Priority | File | Action |
|----------|------|--------|
| P0 | `src/bmm-skills/3-solutioning/bmad-create-epics-and-stories/workflow.md` | Add `State Recovery Check` section (per `fixes/evolution-2/workflow-gate-patch-template.md`) |
| P0 | `src/bmm-skills/3-solutioning/bmad-create-epics-and-stories/steps/step-01b-continue.md` | NEW FILE (use `fixes/evolution-2/step-01b-continue-epics.md`) |
| P0 | `src/bmm-skills/3-solutioning/bmad-create-epics-and-stories/steps/step-01-validate-prerequisites.md` | Add Section-0 "Check for Existing Workflow State" guard |
| P1 | `src/bmm-skills/3-solutioning/bmad-check-implementation-readiness/workflow.md` | Add `State Recovery Check` section |
| P1 | `src/bmm-skills/3-solutioning/bmad-check-implementation-readiness/steps/step-01b-continue.md` | NEW FILE (use `fixes/evolution-2/step-01b-continue-readiness.md`) |
| P1 | `src/bmm-skills/3-solutioning/bmad-check-implementation-readiness/steps/step-01-document-discovery.md` | Add Section-0 guard |
| P1 | `src/bmm-skills/3-solutioning/bmad-generate-project-context/workflow.md` | Add `STATE RECOVERY CHECK` section before EXECUTION |
| P1 | `src/bmm-skills/3-solutioning/bmad-generate-project-context/steps/step-01b-continue.md` | NEW FILE (use `fixes/evolution-2/step-01b-continue-project-context.md`) |
| P1 | `src/bmm-skills/3-solutioning/bmad-generate-project-context/steps/step-01-discover.md` | Add Section-0 guard |
| P2 | `src/bmm-skills/4-implementation/bmad-sprint-planning/workflow.md` | Add State Recovery preamble (XML-style for workflow-only skill) per `fixes/evolution-2/yaml-workflow-state-recovery.md` |
| P2 | `src/bmm-skills/4-implementation/bmad-create-story/workflow.md` | Add `<step n="0">` state recovery gate |
| P2 | `src/bmm-skills/2-plan-workflows/bmad-create-product-brief/workflow.md` and 3 other already-have-step-01b skills | Add explicit `State Recovery Check` section to make the gate authoritative (currently relies on step-01-init implicit routing) |
| P3 | `src/bmm-skills/2-plan-workflows/bmad-validate-prd/workflow.md` + step-01b file | New target — multi-step persistent output |
| P3 | `src/bmm-skills/4-implementation/bmad-code-review/workflow.md` + step-01b file | New target |
| P3 | research workflows (3) | New target |
| P3 | `src/bmm-skills/4-implementation/bmad-quick-dev/workflow.md` + step-01b file | New target |

### 5.3 Evolution-3 (status registry write-back) — install patches

Every completion step needs the patch block from
`fixes/evolution-3/workflow-completion-writeback-patch.md`.

| Priority | File | Action |
|----------|------|--------|
| P0 | `src/bmm-skills/3-solutioning/bmad-create-epics-and-stories/steps/step-04-final-validation.md` | Add Workflow Status Registry Update block (registry name `create-epics-and-stories`) |
| P0 | `src/bmm-skills/2-plan-workflows/bmad-create-prd/steps-c/step-12-complete.md` | Replace generic "update workflow_status (if exists)" with explicit patch (registry name `prd`); make unconditional |
| P0 | `src/bmm-skills/2-plan-workflows/bmad-create-ux-design/steps/step-14-complete.md` | Same — registry name `create-ux-design` |
| P0 | `src/bmm-skills/3-solutioning/bmad-create-architecture/steps/step-08-complete.md` | Add explicit patch (registry name `architecture`) |
| P0 | `src/bmm-skills/1-analysis/bmad-create-product-brief/steps/step-06-complete.md` | Make unconditional, add path resolution + template auto-create |
| P1 | `src/bmm-skills/3-solutioning/bmad-check-implementation-readiness/steps/step-06-final-assessment.md` | New target — registry name `solutioning-gate-check` |
| P1 | `src/bmm-skills/3-solutioning/bmad-generate-project-context/steps/step-03-complete.md` | New target |
| P1 | `src/bmm-skills/4-implementation/bmad-sprint-planning/workflow.md` (terminal) | New target |
| P2 | `src/bmm-skills/2-plan-workflows/bmad-validate-prd/steps-v/step-v-13-report-complete.md` | New target |
| P2 | `src/bmm-skills/1-analysis/research/bmad-domain-research/domain-steps/step-06-research-synthesis.md` | New target |
| P2 | `src/bmm-skills/1-analysis/research/bmad-technical-research/technical-steps/step-06-research-synthesis.md` | New target |
| P2 | `src/bmm-skills/1-analysis/research/bmad-market-research/steps/step-06-research-completion.md` | New target |
| P2 | `src/bmm-skills/4-implementation/bmad-correct-course/workflow.md` (terminal) | New target |
| P3 | `src/bmm-skills/4-implementation/bmad-retrospective/workflow.md` | New target |
| P3 | `src/bmm-skills/4-implementation/bmad-code-review/steps/step-04-present.md` | New target |
| (template) | `~/.claude/config/bmad/templates/bmm-workflow-status.template.yaml` | Add `create-epics-and-stories` entry (Fix D3) — outside this repo |
| (template) | `~/.claude/commands/bmad/workflow-status.md` | Add fallback + reconciliation-on-read (Fix D1) — outside this repo |
| (template) | `~/.claude/agents/bmad-master*` | Add post-workflow completion verification hook (Fix D2) — outside this repo |

### 5.4 Evolution-4 (mandatory artifact) — install patches

| Priority | File | Action |
|----------|------|--------|
| P0 | `src/bmm-skills/4-implementation/bmad-dev-story/workflow.md` | Step 1 `<action if="story file inaccessible">` currently HALTs — change to "CREATE artifact at canonical path using prior-story template" per E4 wording. Step 10 `<action>Update File List section…</action>` block — add MANDATORY artifact-create-or-update directive (mirror the slash-command patch). |
| P1 | `src/bmm-skills/4-implementation/bmad-create-story/workflow.md` | Make output path/sections MANDATORY; remove any `if exists` conditionals on the artifact file. |
| P1 | All E3-affected completion steps | When applying E3 patch, replace any "update workflow status (if exists)" with the E4-form: "MANDATORY. If exists: update. If not exists: create from template." (eliminates the `do X (if Y)` defect class identified in E4 generalization) |
| P2 | `src/bmm-skills/4-implementation/bmad-quick-dev/step-oneshot.md` | Audit one-shot path for the same artifact-skip defect |
| P2 | `src/bmm-skills/4-implementation/bmad-quick-dev/step-05-present.md` | Audit final step for artifact creation |

---

## Section 6 — Summary

| Evolution | Original target count | Applied (full) | Applied (partial) | Absent | New skills to add |
|-----------|----------------------|----------------|-------------------|--------|-------------------|
| E1 | 5 (architecture, ux-design, technical-research, readiness, correct-course) | 3 (architecture, ux-design, technical-research) | 0 | 2 (readiness, correct-course) | 5 (validate-prd, epics-and-stories, generate-project-context, create-story, quick-dev) |
| E2 | 5 (epics, readiness, project-context, sprint-planning, create-story) | 0 | 0 | 5 | 6 (validate-prd, code-review, 3 research, quick-dev) |
| E3 | 8 named (with registry mention) | 0 | 8 (mention generic "workflow status" — none use registry path/template) | varies | 6+ (readiness, project-context, sprint-planning, validate-prd, retrospective, code-review, correct-course) |
| E4 | 1 (slash-command, outside this repo) | 0 in this repo (skill workflow has same latent defect) | 0 | 1 (skill `bmad-dev-story/workflow.md`) | 3 (create-story, quick-dev variants) |

**Net assessment:** Evolution-1 is materially applied for the deliberative
authoring skills (architecture, ux-design, prd, product-brief, research) but
NOT for the readiness/correct-course validation skills the fix files were
authored for. Evolution-2 is partially seeded (4 step-01b files exist with
correct semantics) but the workflow.md gate is missing in all skills, and
the original 5 P0/P1/P2 targets received nothing. Evolution-3 has only
generic "workflow status" prose with no registry path/template logic — the
spec is essentially unimplemented in `src/`. Evolution-4 was applied to the
slash-command (in `~/.claude/commands/bmad/dev-story.md`, outside this
repo) but the same defect remains in the skill workflow at
`src/bmm-skills/4-implementation/bmad-dev-story/workflow.md`.

The largest leverage upstreaming move is the Evolution-2 P0 (epics-and-stories)
+ Evolution-3 P0 set, which together close the
"workflow runs → no state recovery → no registry update → dashboard blind" gap
that motivated both fixes.
