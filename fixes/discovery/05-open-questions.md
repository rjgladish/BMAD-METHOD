# 05 — Open Questions for User

> Questions to resolve BEFORE Phase 6 execution begins. The answers shape Wave 2 and 3 design materially.

---

## Q3 (carried from P-0) — Template direction for state-tracking substrate

User said: "some versions of bmad used yaml to track state, others used markdown."

The catalogue confirms BMAD's own oscillation: `bmm-workflow-status.yaml` (registry) coexists with `stepsCompleted` markdown frontmatter (per-document state) coexists with `<step n="N">` ordinals in `instructions.xml` (FSM-as-prose).

**To resolve:** Which substrate should Wave 2 (state machinery) consolidate on?

- **(a) Schema-validated YAML registry as source of truth.** Workflows write through a validator. Step files become pure renderers against the registry. Markdown `stepsCompleted` is removed; XML `<step n="N">` ordinals are removed. State is data, not control flow. Cost: substantial refactor of every multi-step workflow's step files.
- **(b) Declarative state machine in YAML.** The FSM is named explicitly (states + transitions). Workflows are state-transition handlers. Cost: introduces a new abstraction (state machine spec) that doesn't yet exist in BMAD.
- **(c) Both — schema OF a declared FSM.** The state machine spec is governed by a LinkML-style schema; instances are validated. Highest discipline, highest implementation cost.
- **(d) Defer.** Keep current dual representation; only fix Evolution-3 writeback for the YAML registry.

Recommended for first pass: **(a)** — it gives the largest wins for the smallest substrate change and is the direction the architecture-repo extracted-YAML pipeline already validates as workable.

A3 - a
## Q4 — Python rewrite scope for utility scripts

User said: "scripts (perhaps rewritten in python)" in the kickoff.

`/home/randyg/repos/architecture/elt/extract-section.sh` is bash. Per linkml memory, "Parsers in `~/repos/architecture/tools/parsers/`" exist (likely additional language not yet inspected).

**To resolve:** Which scripts get rewritten as Python utilities promoted into BMAD upstream?

- **(a) Just `extract-section.sh`** — narrow, one-purpose, low cost.
- **(b) The full `tools/parsers/` family** — adoc-type-aware parsers (UC, US, SS, DM, AC, ConOps).
- **(c) Plus the Jinja2/Mustache `lkml-gen` templates** — round-trip YAML ↔ adoc, with templates as data.
- **(d) Plus a LinkML schema for the requirements metamodel** — eventual governance.

Each level is a wave's worth of work. Phase 6 sequencing depends on the answer.

A4 - Option e - a LinkML metamodel for each YAML schema used to generate an adoc, plus a lkml gen template (note your usage is out of date lkml-gen is no longer a tool, all cli capabilities accessed via lkml tool).

We can defer the extraction (or eliminate) with a different flow proposed YAML generates adoc for review, changes made to proposed YAML, then adjudicated.
---

## Q5 — Rebase strategy for `enhance/better-adr-capture`

User kickoff said: "rebase with main (merge as necessary)."

`main` has moved substantially since this branch diverged (28+ commits include the `9725b0ae refactor(skill)` flatten, `25d24d02` dev-story to native skill, `0380656d refactor: consolidate agents into phase-based skill directories #2050`).

**To resolve:**
- **(a) Rebase enhance onto main** — replay the 4 enhance commits onto the new main HEAD. Cleaner history; conflicts must be resolved one-by-one.
- **(b) Merge main into enhance** — merge commit, all main changes appear together. Preserves enhance commit dates.
- **(c) Cherry-pick the substantive commits, drop the rest** — most surgical.

Most of `enhance/better-adr-capture`'s 4 commits add files under `fixes/` (no conflict potential with `src/`). The real conflict surface is the dev-story changes. Recommended: **(a) rebase**, with the explicit understanding that all upstream evolution work happens AFTER the rebase against the new skills tree, not against the older paths in the fix files.

A5 - a

---

## Q6 — Evolution-1 file-name reconciliation

The architecture skill in `src/` has files like `step-03-starter.md`, `step-04-decisions.md` whereas Evolution-1's fix files are named `step-03-core-experience.md`, `step-04-architectural-patterns.md`.

**To resolve:**
- Are the upstream renames intentional (the workflow shape changed and the new names supersede the old)?
- Or is Evolution-1's content meant to overwrite the upstream file content under whatever new names are canonical?

Recommended: ride upstream naming; map Evolution-1's INTENT (FACILITATOR / A/P/C / stepsCompleted / FORBIDDEN-to-load-next-step / NEVER-generate-without-input) onto whatever the file name happens to be. The Evolution-1 fix files in `fixes/evolution-1/` then become **historical evidence**, not patch-ready content.

**A6** - I do not recall/know the origin the renaming, that may have been a claude thing.
---

## Q7 — Scope of "state-machine sickness" remediation

The smoking gun is corpus-wide ("LOAD the FULL workflow.md, READ its entire contents and follow its directions exactly!" appears across 8+ projects). The FSM-in-prose is intrinsic to BMAD agent files (e.g., `bmad-master.md`'s `<activation critical="MANDATORY">` block with `<step n="1..9">` ordinals).

**To resolve scope:** is the goal in Wave 2 to
- **(a) Convert workflow step files only** — leave agent activation FSMs alone. Lower risk, smaller wins.
- **(b) Convert workflow step files AND agent activation blocks** — comprehensive. Requires re-architecting how agents declare their lifecycle.
- **(c) Pilot on ONE workflow + ONE agent** — prove the pattern, defer corpus-wide rollout to a future evolution.

Recommended: **(c) pilot** — `bmad-create-architecture` (workflow) + `bmad-master` (agent) as the two pilots, since they're the most exercised in the user's projects.

**A7** - Incremental. Do a then c. I have yet to fully committed to BMAD. Much of the verbiage of BMAD is a heritage from less capable LLM, it is highly likely that the bare bones steps are much more simple and straight-forward with verifiable gate tests and FSM logic implemented a a proper state machine.

It is hard to separate the annoying ceremonial aspects of BMAD from the extra decorations required for it to behave, LLMs are vert poor state machines (i.e. a non-deterministic state machine == gambling)

---

## Q8 — Scope of "AsciiDoc + YAML + scripts" promotion

The architecture-repo pipeline (adoc with stable IDs → extract-section.sh → YAML → Jinja templates → adoc/markdown) is a working substrate that **already exists outside BMAD**. The question is how much of it BMAD should ABSORB vs. just RECOGNIZE.

- **(a) Recognize only.** BMAD documents the pattern in a new skill (`bmad-asciidoc-substrate`) but does not ship parsers/templates. Projects keep their own.
- **(b) Absorb parsers + templates.** BMAD ships canonical Python parsers and Jinja templates for UC/US/SS/DM/AC/ConOps; projects use them via skill invocation.
- **(c) Absorb the whole metamodel.** BMAD ships a LinkML schema governing the requirements/UC/US/SS/DM/AC types; parsers/templates derive from the schema; project YAML files validate against it.

Recommended: start with **(b)** — concrete, immediately useful, doesn't require LinkML adoption first. Schedule **(c)** as a follow-on once **(b)** ships.

**A8** - We need a migration path. We can't abandon the extraction scripts, but they can be deprecated - use in case of authoritative adocs. I have not decided whether the YAML as exclusively authoritative is yet feasible (pragmatic).

Most BMAD users are probably unfamiliar with asciidoc, and don't care about the rigor, but serious projects need the features it offers. If I continue with BMAD, it will likely be a custom implementation, because the integration with IDEs and dashboards are terrible, no doubt exacerbated by the limitations exposed in this study.
---

## Q10 — Promote the review-edit-extract-diff loop as a canonical BMAD pattern?

User pointer (2026-04-29) flagged: *"the adoc review + edit + YAML extraction + diff is a very good change proposal loop that's worth noting. This matches similar agent/operator collaboration cycles."*

The loop puts operator at the authoring seat, agent at the extraction seat, and the YAML diff at the center as the canonical communication artifact. It matches the Evolution-1 "FACILITATOR not content generator" intent, but expresses it as a workflow primitive rather than a discipline rule.

**To resolve:** Should this loop become a first-class BMAD pattern in Wave 2/3?

- **(a) Document only.** Note the pattern in BMAD documentation; no skills, no enforcement.
- **(b) Codify as a skill.** New skill `bmad-review-extract-diff` (or extend `bmad-correct-course`) that wraps the loop: invoke extractor, diff against baseline YAML, present diff, accept/reject/iterate.
- **(c) Make it the default for ALL Wave 3 artifact-discipline work.** Every "C → continue" transition produces a YAML diff for operator review. Replaces or augments today's opaque "save" action. Higher discipline, requires every workflow's output to have a derivable YAML form.
- **(d) Defer.** Capture as substrate aspiration; address in a future evolution.

Recommended: **(b) for Wave 2 + selective (c) on the high-value workflows** (PRD, architecture, story creation, dev-story). The pattern is most valuable where the artifact has high downstream consumption.

A10 e - An agile process consume the available material. We should allow markdown, YMAL, or asciidoc to enter the bmad system as authoritative, then is normalized and curates ground truth, then generate derivatives to facilitate change proposals, review, and revision. Once accepted, YAML remain the ground truth, then change proposals and review derivatives can be generated/approved. YAML is simply easier to validate and version manage at the business object level, where are formatted texts is simply harder to mechanistically govern and align.

The use of PAV to document the relationship should be normative.

---

## Q12 — Registry schema redesign: Wave 1 or Wave 2?

User-supplied evidence (2026-04-29): inspected `bmm-workflow-status.yaml` instances in `luka/`, `teague/`, `linkml/`, `kai/Elements/`. The schema has drifted across projects — kai/Elements adopted **flat top-level keys with list-valued `prd:`** to handle multi-output projects; the other three keep the original `workflow_status:` list and can't represent multi-PRD projects. 8 deficiencies catalogued (RS-1 through RS-8 in `00-prior-context.md`).

**The registry schema and the requirements YAML schema (FR-1..FR-8) are the same class of substrate problem.** Both lack subsystem prefixes, both overload single fields with multiple roles, both lose rationale, both lack diff-friendly canonical form.

**To resolve:**

- **(a) Wave 1 redesign — governed schema, applies to both registry and requirements YAML.** Define a single LinkML metamodel that covers workflow-status entries AND requirements/UC/US/SS/DM/AC entries. Both are "structured artifacts with traces, rationale, lifecycle." Cost: requires LinkML adoption (or equivalent) before any Wave 2/3 work; biggest substrate change.
- **(b) Wave 1 schema redesign for registry alone, defer requirements unification to a follow-on wave.** Address RS-1..RS-8 as a Wave 1 deliverable; let FR-1..FR-8 wait for Wave 4+. Cost: two passes through the schema work, but unblocks E3 writeback for Wave 2.
- **(c) Wave 2 — keep the existing registry schema, just implement the writeback machinery.** RS-1..RS-8 stay open; the kai/Elements multi-PRD case keeps its bespoke schema; new projects continue to drift. Cost: lowest immediate work, but Evolution-3 ships against a known-broken schema.
- **(d) Decoupled — registry redesign and requirements YAML redesign as TWO independent Wave 1 efforts.** Each has its own scope; shared LinkML governance is a Wave 4 aspiration. Cost: middle ground; risk of schema-design divergence between the two efforts.

Recommended: **(a)** if LinkML adoption is acceptable as a Wave 1 prerequisite; otherwise **(b)**. **(c) is rejected** — shipping E3 against the broken registry schema repeats the Evolution-3 mistake (the README's "Fix D3: add missing entries" was already a band-aid; doing more band-aids is not the path).

**Connection to Q11:** if Q11 closes all 8 FR deficiencies in Wave 1 AND Q12 picks (a) or (b) for the registry, both efforts converge on a unified metamodel. If Q11 picks a subset and Q12 picks decoupled (d), the two schemas drift from inception.

A12

Do not get too locked into a unified LinkML metamode coverying registry+requirements. The issue is broader than requirements. The AND is signal that composition may be indicated. Why is one bettter than multiple? Are you mixing concerns?

The drift occurred because I was tired of explaining to Claude/BMAD why their convention was deficient, the preferred method for requirements identifiers is enterprise-wide unique and hierarchical. Good requirements engineering does not depend on the project.



The schema used in elt and fw are probably the closest to ideal, but the CSV generated for review was useful. A LinkML model that exposed the spreadsheet column names mapped to the YAML schema, while preserving the fidelity of the asciidoc source is likely to be the big win.

The BIG win comes from traceability between the derived artifacts. A project level document (LinkML) should specify the identifier schema (this is present in elt and fw, but it is semantically weaker than it should be.) If this were combined with CALM or C4 Architecture artifact, recall CALM metamodel work, this creates traceability across CONOPS, requirements, user stories, use cases, and architecture. A similar discipline for quality assurance, prompt engineering (BAML), and other lkml 'expansion packs' and consisent use of PAV discipline would link lifecycle artifacts.

---

## Q11 — FR-extraction-exposed deficiencies: include in Wave 1 substrate scope?

The 8 FR-extraction deficiencies (FR-1 through FR-8 in `00-prior-context.md`) span numbering convention, schema fidelity, cross-reference preservation, traceability, hierarchy, and versioning. They are NOT downstream sloppiness; they are gaps in the canonical substrate.

**To resolve:** Is Wave 1 (substrate) responsible for closing all 8, or only some?

- **(a) All 8.** Canonical schema includes subsystem-prefixed IDs, full `{id, title, rationale, source, traces_to, phase, strength, test_procedure}`, cross-reference mechanics, traceability matrix, capability-area hierarchy, versioned acceptance criteria.
- **(b) The "fidelity" subset (FR-3, FR-5, FR-7, FR-8)** — schema fields and structural relationships. Defer numbering convention (FR-1, FR-2) and traceability matrix (FR-6) to a follow-up wave.
- **(c) Pilot subset (FR-1, FR-2, FR-3)** — the most impactful deficiencies; prove the substrate change works before expanding.

Recommended: **(a)** — the deficiencies are tightly coupled. Solving FR-3 (schema fields) without FR-1/FR-2 (numbering) leaves the canonical schema with bad IDs; solving FR-7 (hierarchy) without FR-4 (cross-references) leaves dangling links. Closing all 8 in Wave 1 is the minimum coherent change.

A11 - See above this must begin with a project-level initialization and GOVERNANCE of a LinkML compatible document (YAML, JSON, LinkML), with authoritative sources clearly specified, and meticously curated, none of this slop dropping files whereever, and then languishing.

This is required regardless of whether it is BMAD, GSD, or pro-workflow
---

## Q9 — ADR rationale capture (the original `enhance/better-adr-capture` mission)

The branch name suggests its first-class purpose was ADR-rationale capture — which the corpus confirms is exactly the failure mode in the architecture-repo extraction pipeline (rationale missing from `extracted/*.yaml` schema even though the proposed schema had it).

**To resolve:** Is "better ADR capture" a Wave 1 deliverable, or the entire purpose of the branch with the broader evolution work as supporting infrastructure?

Recommended interpretation: **better ADR capture is Wave 1's headline deliverable** (D2), implemented as: (a) ADR canonical shape (Status / Context / Decision / Consequences / Rationale / Links); (b) YAML schema includes `rationale` field as required; (c) `architecture-decision-template.md` updated; (d) parsers (when promoted in Q8) extract rationale.

**A9** - yes