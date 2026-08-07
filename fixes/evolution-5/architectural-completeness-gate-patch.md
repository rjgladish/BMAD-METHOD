# Patch: Architectural-Completeness Gate for `bmad-bmm-create-story`

**Files:**
- **Canonical source:** `src/bmm-skills/4-implementation/bmad-create-story/workflow.md`
- **Project-installed mirror:** `_bmad/bmm/workflows/4-implementation/create-story/instructions.xml`
- **Checklist:** `src/bmm-skills/4-implementation/bmad-create-story/checklist.md` (add §6)
- **Template:** `src/bmm-skills/4-implementation/bmad-create-story/template.md` (replace "Out of Scope" section shape)

**Insertion site:** Between current `<step n="5">` (Create comprehensive story file) and `<step n="6">` (Update sprint status and finalize), insert a new `<step n="5b">` that runs the gate AFTER the file is drafted but BEFORE `ready-for-dev` is set and before sprint-status is updated.

**Applied:** PENDING

---

## Why a step-5b gate (not absorbed into step-5 or step-6)

- Step-5 generates content; step-6 commits status. The gate is an audit.
  Audits between author-and-commit is the standard place for self-review.
- Step-5 already sets `Status: ready-for-dev` in the template body
  (`<action>Set story Status to: "ready-for-dev"</action>`). The gate must
  run BEFORE that line; otherwise the workflow has already promoted the
  story by the time the audit fires.
- A separate step keeps the gate independently editable, independently
  skippable for known-orthogonal stories, and independently auditable in
  evolution-audit scans (analogous to how Evolution-2's state-recovery
  gate is a discrete `<step n="0">`).

**Implementation note for step 5:** The
`<action>Set story Status to: "ready-for-dev"</action>` line MUST be MOVED
from step 5 to step 6 (after the gate passes). Until step 5b passes, status
remains `draft`.

---

## Before (defective — workflow.md, lines 293-345)

```xml
<step n="5" goal="Create comprehensive story file">
  <critical>📝 CREATE ULTIMATE STORY FILE - The developer's master implementation guide!</critical>

  <action>Initialize from template.md: {default_output_file}</action>
  <template-output file="{default_output_file}">story_header</template-output>

  <!-- Story foundation from epics analysis -->
  <template-output file="{default_output_file}">story_requirements</template-output>

  <!-- ... template-output blocks ... -->

  <!-- CRITICAL: Set status to ready-for-dev -->
  <action>Set story Status to: "ready-for-dev"</action>
  <action>Add completion note: "Ultimate
  context engine analysis completed - comprehensive developer guide created"</action>
</step>

<step n="6" goal="Update sprint status and finalize">
  <action>Validate the newly created story file {default_output_file} against `./checklist.md` and apply any required fixes before finalizing</action>
  <action>Save story document unconditionally</action>
  ...
</step>
```

The defect: nothing between content authoring and status promotion checks
whether the story FULLY MANIFESTS its bound architectural contract.
Internal-coherence validation (`./checklist.md`) runs in step 6 but only
catches story-shape defects, not architectural-completeness defects.

## After (fixed)

```xml
<step n="5" goal="Create comprehensive story file">
  <critical>📝 CREATE ULTIMATE STORY FILE - The developer's master implementation guide!</critical>

  <action>Initialize from template.md: {default_output_file}</action>
  <template-output file="{default_output_file}">story_header</template-output>

  <!-- Story foundation from epics analysis -->
  <template-output file="{default_output_file}">story_requirements</template-output>

  <!-- ... template-output blocks ... -->

  <!-- DO NOT set status here. Status remains 'draft' until step 5b passes. -->
  <action>Set story Status to: "draft"</action>
  <action>Add completion note: "Story draft complete - awaiting architectural-completeness gate"</action>
</step>

<step n="5b" goal="Architectural-completeness gate (REQUIRED before ready-for-dev)">
  <critical>🏛️ ARCHITECTURAL-COMPLETENESS GATE — A story bound to a binding ADR / spec / epic / PRD section MUST FULLY MANIFEST that contract before advancing to ready-for-dev. Deferral of architectural manifestation work to "follow-up stories" is a recurring defect that produces internal contradiction and architectural debt. See evolution-5/scope-management-defect-pattern.md.</critical>

  <action>Execute the 5-question architectural-completeness self-audit and record verbatim answers in the story file under a new section titled "Architectural Completeness Audit" (placed just before "Out of Scope"):</action>

  <gate-question n="1" id="binding-contract-identification">
    <prompt>Does this story have a binding ADR / spec / epic / PRD section?</prompt>
    <action>Scan the drafted story for citations of ADRs, specs, epics, PRD sections that the story claims to apply, instantiate, manifest, or implement. Look for phrases: "per ADR-NN", "per [spec]", "per epic [N]", "first application of", "worked example of", "manifests", "implements [contract]".</action>
    <check if="no binding contract cited">
      <action>Record answer: "n/a — no binding contract identified"</action>
      <action>Skip questions 2-5</action>
      <action>GATE RESULT: PASS (n/a)</action>
    </check>
    <check if="binding contract cited">
      <action>Record verbatim: contract name (e.g., "ADR-27"), contract path, the specific clause(s) the story claims to apply, and the contract's mandated work for this surface (quote the contract text)</action>
      <action>Proceed to question 2</action>
    </check>
  </gate-question>

  <gate-question n="2" id="manifestation-completeness">
    <prompt>Does this story's scope FULLY manifest the binding contract's mandated work for this surface?</prompt>
    <action>Compare the contract's mandated work (recorded in Q1) against the story's Tasks/Subtasks. List any mandated work item that does NOT appear in scope.</action>
    <check if="any mandated work item is absent from scope">
      <action>For each absent item, identify where it is deferred to (follow-up story, separate ADR, "future phase", "next epic", "tracked separately")</action>
      <action>Record: "DEFERS — [list items, where deferred]"</action>
      <action>RAISE FLAG Q2-DEFERS</action>
    </check>
    <check if="all mandated work present">
      <action>Record: "FULL — all contract-mandated work present in scope"</action>
    </check>
  </gate-question>

  <gate-question n="3" id="out-of-scope-architectural-audit">
    <prompt>For each "Out of Scope" entry, classify as (a) orthogonal, (b) future enhancement, or (c) DEFERRED ARCHITECTURAL MANIFESTATION of the bound contract.</prompt>
    <action>For each Out-of-Scope entry, determine whether the binding contract identified in Q1 mandates this work for this surface. If YES → classification (c) RED FLAG. If the work is unrelated to the bound contract → (a). If the work extends the bound contract beyond what is mandated for this surface → (b).</action>
    <action>Record classification table: [entry text] → (a|b|c) → [justification]</action>
    <check if="any classification is (c)">
      <action>RAISE FLAG Q3-DEFERRED-ARCH</action>
    </check>
  </gate-question>

  <gate-question n="4" id="internal-contradiction-sweep">
    <prompt>Does the story's prose claim to be "first application", "worked example", "exemplar", or "manifestation" of the bound contract while the scope does not actually do that?</prompt>
    <action>Grep the drafted story for self-framing phrases: "first application of", "worked example", "exemplar", "manifests the pattern", "implements the [contract] design", "demonstrates [contract]", "the canonical instance of"</action>
    <action>For each match, verify the scope (Tasks/Subtasks/AC) actually accomplishes what the prose claims.</action>
    <check if="any self-framing claim is unsupported by scope">
      <action>Record: "CONTRADICTION — [phrase] at [section] vs scope [missing items]"</action>
      <action>RAISE FLAG Q4-CONTRADICTION</action>
    </check>
    <check if="all self-framing claims supported by scope">
      <action>Record: "CONSISTENT — self-framing matches scope"</action>
    </check>
  </gate-question>

  <gate-question n="5" id="rube-goldberg-signature-check">
    <prompt>Does the story propose postprocessor strippers, compat shims, hand-authored bridges duplicating codegen, paperwork-only manifests at one location cross-referencing files at a contract-rejected location, or "open a follow-up to fix the generator" patterns?</prompt>
    <action>Grep the drafted story for signatures: "postprocess", "strip", "shim", "bridge", "wrapper around", "hand-author", "manually maintain", "open a follow-up to fix", "subsequent story to fix the [tool]", "workaround until [tool] is fixed", manifest cross-references where manifest path and member paths disagree on root location.</action>
    <action>For each signature match, evaluate: is this the right tool/placement, or a bandaid for using the wrong tool/placement?</action>
    <check if="any bandaid signature found">
      <action>Record: "BANDAID — [signature] at [section]: [bandaid vs root-cause analysis]"</action>
      <action>RAISE FLAG Q5-BANDAID</action>
    </check>
    <check if="no bandaid signatures">
      <action>Record: "CLEAN — no Rube-Goldberg signatures found"</action>
    </check>
  </gate-question>

  <action>Compute gate result based on raised flags:</action>
  <check if="any of Q2-DEFERS, Q3-DEFERRED-ARCH, Q4-CONTRADICTION, Q5-BANDAID raised">
    <action>GATE RESULT: BLOCK</action>
    <action>Status remains "draft" — do NOT advance to ready-for-dev</action>
    <action>Append "Gate Block Reason" section to story listing flags + remediation paths:
      - Option A: expand scope to fully manifest the bound contract
      - Option B: change story framing — remove "first application of [contract]" claims and explicitly justify the deferred scope against the contract text
      - Option C: change the bound contract — open a contract-revision conversation if the contract's mandate is genuinely impractical for this surface
    </action>
    <output>🚫 ARCHITECTURAL-COMPLETENESS GATE: BLOCKED

      Story remains in 'draft' status. Flags raised:
      {{list of flags}}

      Remediation options written to story under "Gate Block Reason".
      Do NOT proceed to dev-story until gate passes.
    </output>
    <action>HALT — story remains draft, sprint-status NOT updated, do NOT execute step 6</action>
  </check>
  <check if="no flags raised">
    <action>GATE RESULT: PASS</action>
    <action>Append "Architectural Completeness Audit: PASS" to the story's audit section with timestamp</action>
    <action>Proceed to step 6</action>
  </check>
</step>

<step n="6" goal="Update sprint status and finalize">
  <critical>This step ONLY executes if step 5b GATE RESULT was PASS or n/a.</critical>
  <action>Set story Status to: "ready-for-dev"</action>
  <action>Validate the newly created story file {default_output_file} against `./checklist.md` (including new §6 Architectural Completeness) and apply any required fixes before finalizing</action>
  <action>Save story document unconditionally</action>
  ...
</step>
```

---

## Companion patch — `checklist.md` §6 "Architectural Completeness"

Insert as a new section in `src/bmm-skills/4-implementation/bmad-create-story/checklist.md`:

```markdown
## **🏛️ Architectural Completeness Audit (REQUIRED)**

**Critical:** This audit is what evolution-5 added. It runs as gate `<step n="5b">` in the workflow. The checklist version below is the FRESH-CONTEXT validator's parallel pass — both must agree.

For each story being validated, walk these 5 questions and record verbatim answers:

### **6.1 Binding-Contract Identification**

- Does the story cite a binding ADR / spec / epic / PRD section?
- If NO: record `n/a`, skip 6.2-6.5.
- If YES: name contract, link path, quote the contract's mandated work for the surface this story addresses.

### **6.2 Manifestation Completeness**

- List every contract-mandated work item.
- For each, is it in scope (Tasks/Subtasks/AC) of THIS story?
- If any item is deferred to a follow-up story / separate ADR / future phase: **RED FLAG Q2**.

### **6.3 Out-of-Scope Architectural Audit**

- For each "Out of Scope" entry, classify:
  - (a) genuinely orthogonal concern
  - (b) future enhancement beyond contract mandate for this surface
  - (c) DEFERRED ARCHITECTURAL MANIFESTATION of the bound contract — **RED FLAG Q3**

### **6.4 Internal-Contradiction Sweep**

- Grep story for self-framing phrases: "first application", "worked example", "exemplar", "manifests", "demonstrates", "canonical instance".
- For each, verify scope actually does what prose claims.
- Any unsupported claim: **RED FLAG Q4**.

### **6.5 Rube-Goldberg Signature Check**

- Grep story for: postprocess, strip, shim, bridge, hand-author, "follow-up to fix the generator", manifests-at-X-cross-ref-Y patterns.
- For each match: is this root-cause fix or symptom patch?
- Any symptom patch: **RED FLAG Q5**.

### **Gate Decision**

- Q2/Q3/Q4/Q5 RED FLAGS: BLOCK — story remains `draft`, append "Gate Block Reason" with remediation options, HALT.
- All clear: PASS — story advances to `ready-for-dev`, audit recorded.
```

---

## Companion patch — `template.md` (replace "Out of Scope" section)

The current template (in the project-installed copy) does not include a
formal "Out of Scope" section, but stories generated by the workflow
commonly add one. Replace any flat "Out of Scope" with the classified
shape below, and add the "Architectural Completeness Audit" section just
above it:

```markdown
## Architectural Completeness Audit

<!-- Filled by step 5b. Required before status can advance to ready-for-dev. -->

- Q1 (binding contract): {{contract_name_or_na}}
- Q2 (manifestation completeness): {{FULL or DEFERS — list}}
- Q3 (out-of-scope classification): see table below
- Q4 (internal contradiction): {{CONSISTENT or CONTRADICTION — detail}}
- Q5 (Rube-Goldberg signatures): {{CLEAN or BANDAID — detail}}
- Gate result: {{PASS | n/a | BLOCK with flags}}

## Out of Scope

<!-- Classify every entry. (c) classifications BLOCK the gate. -->

### Out of Scope (a) — Genuinely Orthogonal Concerns

- [Entry] — [why orthogonal to bound contract]

### Out of Scope (b) — Future Enhancements Beyond Contract

- [Entry] — [contract section being extended; not mandated for this surface]

### Deferred (c) — REQUIRES JUSTIFICATION

<!-- Any entry here triggers a Q3 RED FLAG. Use this only if Q3 finds (c) classifications. Justify against the bound contract's text or remove. -->

- [Entry] — [explicit justification against bound contract or reframing of the story]
```

---

## Why this wording works

1. **Step-5b is independent.** The gate is its own step, not buried inside
   step-5 or step-6. Audit is independently editable, independently
   skippable for stories with no binding contract, independently visible in
   evolution-audit scans.
2. **Status promotion moves to step-6.** The defect was that step-5 set
   `ready-for-dev` BEFORE any architectural-completeness check ran. Moving
   that line to step-6 closes the gap.
3. **Five questions cover the failure surface.** Q1 anchors the audit;
   Q2 catches deferral; Q3 catches Out-of-Scope mis-classification;
   Q4 catches the prose-vs-scope contradiction signature; Q5 catches the
   Rube-Goldberg residue (postprocessors, shims, follow-up-to-fix-the-tool
   patterns).
4. **Block is structured, not prescriptive.** When the gate trips, the
   workflow does not dictate the fix — it presents three remediation
   options (expand scope / change framing / change contract). The author
   chooses; the gate just refuses to advance until one is chosen.
5. **Audit is recorded in the story.** Every gate run leaves a verbatim
   audit section in the story file, so reviewers can re-validate the
   classification without re-running the gate.

## Generalization

The same "do X (MANDATORY) AND verify X is architecturally complete"
shape applies to every authoring workflow that produces an artifact
bound to upstream contracts. See `gate-questions.md` for the
workflow-agnostic form, suitable for copy-paste into create-prd,
create-architecture, create-epics-and-stories.
