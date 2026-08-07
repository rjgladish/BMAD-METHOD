# Evolution-5: Architectural-Completeness Gate for `bmad-bmm-create-story`

**Date:** 2026-05-05
**Status:** SPEC COMPLETE (PATCH PENDING)
**Depends on:** evolution-1 (deliberative discipline), evolution-2 (state recovery gate)

## Problem

The `bmad-bmm-create-story` workflow validates internal coherence — ACs cover
scope, tasks cover ACs, file list is plausible, source citations exist — but
has **no architectural-completeness check**. A story bound to a binding
ADR / spec / epic can advance to `ready-for-dev` while DEFERRING the
architectural manifestation work the bound contract mandates to a "follow-up
story" or "separate ADR." The current workflow detects neither the deferral
nor the internal contradiction it produces.

The shape of the defect: scope-creep prevention masquerading as scope
discipline. What looks like care is evasion. The story passes every existing
gate — internal coherence, AC traceability, BDD format, source citations —
while shipping incoherence at the architectural level.

## Impact

This is not a one-off. **Three named incidents in the lkml project plus a
recurring memory-feedback pattern**:

1. **2026-04-21 — codegen-mining-family intercept.** A story chose
   `lkml gen pydantic` for an L1/L2 target (a placement that ADR-26 was
   subsequently authored to reject), then patched the resulting
   `from pydantic import BaseModel` contamination with a bespoke
   `postprocess_mining_family.py` stripper AND opened a follow-up
   `gen-pydantic-enum-only-emission` story to "fix the generator." Randyg
   intercepted before merge. Memory:
   [feedback_sharding_without_arch_monitoring.md](/home/randyg/.claude/projects/-home-randyg-repos-linkml/memory/feedback_sharding_without_arch_monitoring.md):
   _"better flyswatter when a rolled up newspaper does the job."_

2. **2026-05-05 — opensearch-metamodel ADR-27 first application.** Story
   drafted as _"first application of ADR-27"_ + _"the worked example,"_ but
   the scope DEFERRED physical relocation of the 11 metamodel + spec files
   from `lkml/docs/specifications/opensearch-metamodel/` (a placement that
   ADR-27 Clause 1 expressly rejects) to a "separate follow-up story" — the
   manifest landed at the correct `lkml/assets/` location while the members
   it cross-referenced stayed at the rejected `lkml/docs/` location.
   Internal contradiction visible on the story page; not caught by the
   create-story workflow. Story file:
   [_bmad-output/implementation-artifacts/story-opensearch-adr27-application.md](/home/randyg/repos/linkml/_bmad-output/implementation-artifacts/story-opensearch-adr27-application.md).

3. **Pattern across sessions.** Memory item
   [feedback_scope_vs_architecture_integrity.md](/home/randyg/.claude/projects/-home-randyg-repos-linkml/memory/feedback_scope_vs_architecture_integrity.md)
   names Randyg's stated polarity:
   _"architecture integrity diligence ≫ scope-creep diligence."_
   Established because dev-story compartmentalization repeatedly inverts
   this polarity. The memory exists as REACTIVE intercept guidance —
   _"at every story intercept: 'symptom patch or root-cause fix?'"_ — not
   as a built-in gate in the workflow.

Cumulative cost: every unmonitored shard contributes a micro-hack;
refactoring them out later is the architectural debt that grinds project
maturity to a standstill. Per Randyg, this has been _"a constant nuisance."_

## Root Cause

The form encourages the behavior. Four reinforcing structural pulls in the
current workflow:

1. **"Out of Scope" as a first-class story section.** The template invites
   listing what the story will NOT do, with no taxonomy distinguishing
   _genuinely orthogonal concerns_ from _deferred architectural manifestation_.
   The agent treats both as equally valid scope limits.

2. **Story-sized norm.** The workflow optimizes for a single-context,
   single-PR-shaped artifact. When a binding contract's full manifestation
   doesn't fit that shape, the form's path of least resistance is to defer
   the over-spill rather than reshape the story.

3. **No architectural-completeness check.** Internal-coherence checks pass
   because they only verify _the story is well-formed_, not _the story is
   architecturally complete with respect to its bound contract_.

4. **Status-board optics reward splits.** Two narrow stories register as
   more progress on the dashboard than one correctly-scoped story. The
   workflow has no countervailing signal to prefer the latter.

Existing memory items address this pattern reactively (intercept on
review). The fix moves the check to authoring time, before
`ready-for-dev`.

## Fix Specification

Add an **architectural-completeness gate** as `<step n="5b">` in
`bmad-create-story/workflow.md`, executed AFTER the story file is drafted
(step 5) and BEFORE `ready-for-dev` is set + sprint-status is updated
(step 6). The gate is a structured self-audit the workflow MUST execute.
On any RED-flag trip, the workflow does NOT advance to `ready-for-dev` —
it routes to rework.

### Gate Questions

1. **Binding-contract identification.** Does this story have a binding
   ADR / spec / epic / PRD section? If NO → skip gate (record `n/a`).
   If YES → name it, link it, summarize the mandate verbatim from the
   contract.
2. **Manifestation completeness.** Does the story's scope FULLY manifest
   the binding contract's mandated work for this surface? Or does any
   part of the mandate live in a "follow-up story" / "separate ADR" /
   "next phase"?
3. **Out-of-Scope architectural audit.** For each "Out of Scope" entry,
   classify as: (a) genuinely orthogonal concern, (b) future enhancement
   beyond the bound contract, or (c) DEFERRED ARCHITECTURAL
   MANIFESTATION of the bound contract (RED FLAG).
4. **Internal-contradiction sweep.** Search the story for self-framing
   phrases ("first application of [contract]", "worked example",
   "manifests the pattern", "exemplar"). For each, verify the scope
   actually does what the prose claims.
5. **Rube-Goldberg signature check.** Search the story's tasks/notes for:
   postprocessor strippers, compat shims, hand-authored bridges that
   duplicate codegen output, "open a follow-up to fix the generator"
   patterns, manifests that cross-reference files at a contract-rejected
   location.

### Block Criteria

Any of the following trips the gate to BLOCK:
- Q2 answer is "defers"
- Q3 produces any (c) classification
- Q4 produces any unresolved self-framing contradiction
- Q5 produces any positive signature match

On block: status remains `draft`. Story is reworked to either (a) include
the architectural manifestation work in scope, or (b) explicitly reframe
the story as something OTHER than "first application of [contract]" with
the deferred scope properly justified against the bound contract's text.

## Fix Files

| File | Purpose |
|------|---------|
| `README.md` | This file — root cause analysis |
| `architectural-completeness-gate-patch.md` | Exact before/after patch for the create-story workflow.md (canonical) and `_bmad/bmm/workflows/4-implementation/create-story/instructions.xml` (project-installed) |
| `gate-questions.md` | The gate questions in self-contained form, suitable for copy-paste into create-prd / create-architecture / create-epics-and-stories |
| `scope-management-defect-pattern.md` | Diagnostic essay for future agents — failure-mode signature, citations, structural pull |

## Applied Patches

### PENDING — bmad-create-story workflow

- [ ] `src/bmm-skills/4-implementation/bmad-create-story/workflow.md` — Insert `<step n="5b">` architectural-completeness gate (canonical source)
- [ ] `src/bmm-skills/4-implementation/bmad-create-story/checklist.md` — Add §6 "Architectural Completeness" with the 5 gate questions
- [ ] `src/bmm-skills/4-implementation/bmad-create-story/template.md` — Replace flat "Out of Scope" section with classified shape: `Out of Scope (orthogonal)` + `Out of Scope (future)` + `Deferred (architectural — REQUIRES JUSTIFICATION)`
- [ ] `_bmad/bmm/workflows/4-implementation/create-story/instructions.xml` (project-installed mirror) — Same gate insertion

### Generalization (deferred to separate evolution-5b or absorbed by individual workflow patches)

The same gate applies to every authoring workflow that produces an
artifact bound to upstream contracts. Candidate hosts:
- `bmad-create-epics-and-stories` — epics as a whole defer arch manifestation
- `bmad-create-architecture` — sub-decisions defer mandates from PRD/charter
- `bmad-create-prd` — sections defer commitments made in product brief
- `bmad-quick-dev` — quick-spec path bypasses the gate by construction; explicit opt-out documented

See `gate-questions.md` for the workflow-agnostic form.

## Related Evolutions

| Evolution | Problem | Status |
|-----------|---------|--------|
| Evolution-1 | Deliberative discipline (FACILITATOR posture, A/P/C menus, NEVER-generate-without-input gates) | Applied to authoring skills (architecture, ux-design, prd, product-brief, research) |
| Evolution-2 | Workflow state recovery gate — step-file workflows restart from scratch across sessions | Applied (P0-P2 HIGH) |
| Evolution-3 | Workflow status registry write-back — `bmm-workflow-status.yaml` never updated on completion | Spec complete, patches pending |
| Evolution-4 | dev-story artifact creation mandatory — "if exists" escape hatch removed | Applied to slash-command; structural defect remains in `bmad-dev-story/workflow.md` |
| **Evolution-5** | **Architectural-completeness gate for create-story — paperwork-only stories pass internal-coherence checks while deferring binding-contract manifestation** | **Spec complete, patch pending** |

## Testing

After applying the patch:

1. **Negative case (block expected).** Draft a story declaring "first
   application of [some-ADR]" with an "Out of Scope" entry that defers
   physical artifact relocation the ADR mandates. Run create-story.
   The gate MUST trip on Q2 + Q3(c) + Q4 and refuse to set
   `ready-for-dev`.

2. **Positive case (pass expected).** Draft a story bound to a binding
   ADR with full manifestation in scope, "Out of Scope" entries all
   genuinely orthogonal, no self-framing contradictions, no
   Rube-Goldberg signatures. Gate MUST pass with all 5 questions
   answered cleanly. Status advances to `ready-for-dev`.

3. **No-binding-contract case (n/a expected).** Draft a story with no
   binding ADR / spec / epic citation (a pure refactor, hygiene story,
   etc.). Gate Q1 answers NO → records `n/a` and skips Q2-Q5.

4. **Regression check.** All existing `ready-for-dev` stories in the
   project should pass the gate retroactively, OR be reclassified
   honestly. The 2026-05-05 opensearch-metamodel ADR-27 story is the
   canonical worked example: BEFORE patch it advanced; AFTER patch it
   blocks until the deferral is removed or the framing is changed.
