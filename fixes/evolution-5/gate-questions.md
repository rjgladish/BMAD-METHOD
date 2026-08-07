# Architectural-Completeness Gate — Questions (Workflow-Agnostic)

**Purpose:** The 5-question gate from evolution-5, in self-contained form
for copy-paste into other authoring workflows that produce artifacts bound
to upstream contracts. Decoupled from `bmad-create-story` specifics.

**Use in:** `bmad-create-prd`, `bmad-create-architecture`,
`bmad-create-epics-and-stories`, `bmad-quick-dev` (with documented opt-out
if quick-spec path is intended to bypass), and any future authoring
workflow that produces an artifact claiming to apply / instantiate /
manifest / implement an upstream contract.

**Insertion site:** Between content authoring and status promotion. The
artifact's status MUST NOT advance to a "ready-for-consumption" state
(e.g., `ready-for-dev`, `ready-for-review`, `approved`) until this gate
passes or returns `n/a`.

---

## The 5 Questions

### Q1 — Binding-Contract Identification

Does the artifact cite a binding upstream contract — ADR, spec, epic,
PRD section, charter clause, mandated standard?

Search for citation phrases: _"per ADR-NN"_, _"per [spec]"_,
_"per epic [N]"_, _"first application of"_, _"worked example of"_,
_"manifests"_, _"implements [contract]"_, _"the canonical instance of"_,
_"in accordance with [standard]"_.

- **NO** → record `n/a`, skip Q2-Q5, gate result PASS (n/a).
- **YES** → name the contract, link the path, quote the specific
  clause(s) the artifact claims to apply, and quote the contract's
  mandated work for the surface this artifact addresses.

### Q2 — Manifestation Completeness

Does the artifact's scope FULLY manifest the contract's mandated work for
this surface? Or is any mandated work item deferred to a follow-up
artifact / separate contract / future phase?

For each contract-mandated work item:
- Is it in scope of THIS artifact?
- If deferred: where to, and why?

- **FULL** (all mandated work in scope) → no flag.
- **DEFERS** (any mandated work deferred) → **RED FLAG Q2**.

### Q3 — Out-of-Scope Classification

For each "Out of Scope" entry in the artifact, classify:

- **(a) Orthogonal** — entry is unrelated to the bound contract.
- **(b) Future enhancement** — entry extends the bound contract beyond
  what is mandated for this surface.
- **(c) Deferred architectural manifestation** — the bound contract
  mandates this work for this surface, and the artifact is removing it
  from scope. **RED FLAG Q3**.

Record classification verbatim per entry. Any (c) → block.

### Q4 — Internal-Contradiction Sweep

Grep the artifact for self-framing phrases:
_"first application"_, _"worked example"_, _"exemplar"_, _"manifests"_,
_"demonstrates"_, _"the canonical instance of"_, _"implements the
[contract] design"_.

For each match: does the scope actually do what the prose claims?

- **CONSISTENT** (claims supported by scope) → no flag.
- **CONTRADICTION** (any unsupported claim) → **RED FLAG Q4**.

### Q5 — Rube-Goldberg Signature Check

Grep the artifact for bandaid signatures:
- Postprocessor strippers / output cleaners
- Compat shims / adapter layers around the bound contract's mandated tool
- Hand-authored bridges that duplicate codegen output
- Cross-references where a manifest at one location references members
  at a contract-rejected location
- _"Open a follow-up to fix the [tool/generator]"_ patterns
- _"Workaround until [tool] is fixed"_ patterns

For each match: is this a root-cause fix or a symptom patch around
mis-using a tool the contract designates correctly?

- **CLEAN** (no signatures) → no flag.
- **BANDAID** (any symptom-patch signature) → **RED FLAG Q5**.

---

## Block Criteria

Any of the following raises BLOCK:
- Q2 = DEFERS
- Q3 produces any (c) classification
- Q4 = CONTRADICTION
- Q5 = BANDAID

On BLOCK: artifact status remains `draft`. Append "Gate Block Reason"
with remediation options:

- **Option A: Expand scope.** Include the architectural manifestation
  work in this artifact.
- **Option B: Reframe.** Remove "first application of [contract]" /
  "worked example" claims; explicitly justify the deferred scope
  against the bound contract's text.
- **Option C: Change the contract.** If the contract's mandate is
  genuinely impractical for this surface, open a contract-revision
  conversation (new ADR, spec amendment, charter update). Do not work
  around it silently.

Author chooses. Gate refuses to advance until one is chosen.

---

## Audit Section (record in artifact)

Every gate run leaves a verbatim record in the artifact:

```markdown
## Architectural Completeness Audit

- Q1 (binding contract): {{contract_name_or_na}}
- Q2 (manifestation completeness): {{FULL or DEFERS — list deferred items}}
- Q3 (out-of-scope classification):
  | Entry | Classification | Justification |
  |-------|----------------|---------------|
  | [text] | (a)/(b)/(c) | [why] |
- Q4 (internal contradiction): {{CONSISTENT or CONTRADICTION — detail}}
- Q5 (Rube-Goldberg signatures): {{CLEAN or BANDAID — detail}}
- Gate result: {{PASS | n/a | BLOCK with flags}}
- Timestamp: {{date}}
- (If BLOCK) Gate Block Reason: see remediation options A/B/C below.
```

---

## Workflow-Specific Adaptations

| Workflow | Bound contract examples | Pre-promotion status | Post-promotion status |
|----------|--------------------------|----------------------|------------------------|
| `bmad-create-story` | ADR, epic, PRD section | `draft` | `ready-for-dev` |
| `bmad-create-prd` | Product brief, charter | `draft` | `approved` |
| `bmad-create-architecture` | PRD, ADRs, charter | `draft` | `approved` |
| `bmad-create-epics-and-stories` | PRD, architecture, ADRs | `draft` | `ready-for-story-creation` |
| `bmad-quick-dev` | (typically none — quick-spec path is small-change-only) | `draft` | `ready-for-dev` (gate may return n/a) |

---

## Why this gate (vs reactive review)

Existing memory items
(`feedback_sharding_without_arch_monitoring.md`,
`feedback_scope_vs_architecture_integrity.md`,
`feedback_no_handwritten_when_generable.md`) all describe the same
failure pattern reactively — "intercept on review." Reactive intercept
depends on the reviewer noticing. Multiple incidents have slipped past
reactive review because the artifact passes internal-coherence checks
while shipping incoherence at the architectural level.

Moving the check to authoring time, structured as 5 explicit questions
with verbatim answers recorded, converts a tacit reviewer skill into
a workflow-enforced gate. The reviewer is no longer the only line of
defense.
