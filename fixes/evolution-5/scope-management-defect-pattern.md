# Scope-Management Defect Pattern — Diagnostic Essay

**For:** Future agents reading evolution-5. Read before drafting any
story bound to a binding ADR / spec / epic.

## The Failure-Mode Signature

A story bound to a binding upstream contract is drafted with status
`ready-for-dev`. Its acceptance criteria reference the contract; its
prose calls itself _"first application of [contract]"_ or _"the worked
example."_ But its scope DEFERS the contract's architectural
manifestation work to a follow-up story or "separate ADR." The story
becomes paperwork-only — it declares a manifest, but the manifest
cross-references files at the location the contract rejects. Internal
coherence checks pass. Architectural completeness fails.

The shape: scope-creep prevention masquerading as scope discipline.
What looks like care is evasion.

## Three Named Incidents

**1. 2026-04-21 — codegen-mining-family intercept.** A story chose
`lkml gen pydantic` for an L1/L2 target. ADR-26 (authored as a direct
response to this incident) rejects that placement. The agent saw the
resulting `from pydantic import BaseModel` contamination and patched
it with a bespoke `postprocess_mining_family.py` stripper, then opened
a follow-up `gen-pydantic-enum-only-emission` story to "fix the
generator." Randyg intercepted by intuition before merge.
[feedback_sharding_without_arch_monitoring.md](/home/randyg/.claude/projects/-home-randyg-repos-linkml/memory/feedback_sharding_without_arch_monitoring.md)
quotes the failure: _"better flyswatter when a rolled up newspaper does
the job."_ The right answer was changing tool choice (`lkml gen python`
over `lkml gen pydantic`), not patching the wrong tool's output.

**2. 2026-05-05 — opensearch-metamodel ADR-27 first application.**
Story drafted as _"first application of ADR-27"_ + _"the worked
example."_ But the scope DEFERRED physical relocation of the 11
metamodel + spec files from
`lkml/docs/specifications/opensearch-metamodel/` (a placement ADR-27
Clause 1 expressly rejects) to a "separate follow-up story" — leaving
the manifest at the correct `lkml/assets/` location while the members
it cross-referenced stayed at the rejected `lkml/docs/` location.
Internal contradiction visible on the story page; not caught by the
create-story workflow. Story file:
[story-opensearch-adr27-application.md](/home/randyg/repos/linkml/_bmad-output/implementation-artifacts/story-opensearch-adr27-application.md).

**3. Pattern across sessions.** Memory item
[feedback_scope_vs_architecture_integrity.md](/home/randyg/.claude/projects/-home-randyg-repos-linkml/memory/feedback_scope_vs_architecture_integrity.md)
records Randyg's polarity verbatim: _"architecture integrity diligence
≫ scope-creep diligence."_ The memory exists as REACTIVE intercept
guidance: _"at every story intercept: 'symptom patch or root-cause
fix?'"_ Multiple incidents slipped past reactive review because the
artifact passes internal-coherence checks while shipping incoherence
at the architectural level.

## The Structural Pull

The current `bmad-create-story` workflow encourages this behavior. Four
reinforcing pulls:

**1. "Out of Scope" as a first-class section.** The form invites listing
what the story will NOT do, with no taxonomy distinguishing genuinely
orthogonal concerns from deferred architectural manifestation. Both look
like equally valid scope limits.

**2. Story-sized norm.** The workflow optimizes for a single-context,
single-PR-shaped artifact. When a binding contract's full manifestation
doesn't fit that shape, the form's path of least resistance is to
defer the over-spill rather than reshape the story.

**3. No architectural-completeness check.** Internal-coherence checks
verify the story is _well-formed_, not that it is _architecturally
complete with respect to its bound contract_. The two are different.

**4. Status-board optics reward splits.** Two narrow stories register
as more progress than one correctly-scoped story. No countervailing
signal.

## Related Memory

- [feedback_sharding_without_arch_monitoring.md](/home/randyg/.claude/projects/-home-randyg-repos-linkml/memory/feedback_sharding_without_arch_monitoring.md)
  — _"better flyswatter when a rolled up newspaper does the job."_
- [feedback_scope_vs_architecture_integrity.md](/home/randyg/.claude/projects/-home-randyg-repos-linkml/memory/feedback_scope_vs_architecture_integrity.md)
  — _"architecture integrity diligence ≫ scope-creep diligence."_
- [feedback_no_handwritten_when_generable.md](/home/randyg/.claude/projects/-home-randyg-repos-linkml/memory/feedback_no_handwritten_when_generable.md)
  — Hand-authored bridges that duplicate codegen output are a Q5 RED
  FLAG (Rube-Goldberg signature).
- [feedback_story_scope_needs_epic_context.md](/home/randyg/.claude/projects/-home-randyg-repos-linkml/memory/feedback_story_scope_needs_epic_context.md)
  — Stories intentionally don't carry the whole picture; before
  proposing a "simpler" scope, read the epic, ADR, and plan.

## What Evolution-5 Adds

The reactive-intercept memory items above name the pattern but rely on
reviewer attention. Evolution-5 moves the check to authoring time —
the gate runs as `<step n="5b">` in `bmad-create-story/workflow.md`,
between content authoring (step 5) and status promotion (step 6). Five
structured questions, verbatim answers recorded in the story file,
status remains `draft` if any flag trips. The reviewer is no longer
the only line of defense.

The gate is not prescriptive. When it trips, three remediation options
are offered: expand scope, reframe the story, or change the contract.
The author chooses. The gate just refuses to advance until one is
chosen.
