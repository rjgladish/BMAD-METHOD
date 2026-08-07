---
stepsCompleted: [1, 2]
inputDocuments:
  - fixes/discovery/00-prior-context.md
  - fixes/discovery/01-project-inventory.md
  - fixes/discovery/03-master-catalogue.md
  - fixes/discovery/04-evolution-audit.md
  - fixes/discovery/04-integration-plan.md
  - fixes/discovery/05-open-questions.md
  - ~/.claude/sessions/2026-04-30-bmad-evolution-discovery-session.tmp
  - ~/.claude/projects/-home-randyg-repos-harness-skills-BMAD-METHOD-git/401fc7dc-6056-4554-b07f-2f0f31fe9bab.jsonl
session_topic: 'Generate engaged-judgment next-action alternatives from the L661-stable integration plan, without re-triggering the simplification-vs-sycophant pendulum.'
session_goals: 'Ranked menu of next-moves with: pre-conditions, single concrete first-step, what would invalidate it, approval required. Operating-mode invariants derived from real failure data. Multi-perspective evaluation of top-ranked moves.'
selected_approach: 'ai-recommended'
techniques_used:
  - failure-analysis (deep)
  - decision-tree-mapping (structured)
  - six-thinking-hats (structured)
ideas_generated: []
context_file: 'fixes/discovery/00-prior-context.md'
---

# Brainstorming Session — Calm-Discovery Next-Moves

**Date:** 2026-04-30
**Branch:** `enhance/better-adr-capture`
**Pickup point:** L661-stable state of `04-integration-plan.md` (after user critique L633 was correctly applied; before L670 unprompted self-flagging triggered the sycophant cascade L671–L684 that user terminated at L692).

---

## Session Overview

**Topic:** Given the integration plan in its L661-stable state (Wave 0 governance → Wave 1 substrate with AsciiDoc co-equal → Wave 2 verified Evolutions → Wave 3 FSM research → Wave 4 decomposed → Wave 5 hygiene), generate engaged-judgment next-action alternatives that advance the work without re-triggering the simplification-vs-sycophant pendulum.

**Goals:**
- A ranked menu of next-moves
- Each: pre-conditions, single concrete first-step, what would invalidate, approval required
- Operating-mode invariants derived from the L633→L692 trace as concrete failure data
- Multi-perspective evaluation of top-ranked moves

**Constraint reframing:** the canonical workflow's "100+ ideas" target applies to divergent creative sessions. For a structured ranked-menu outcome, the discipline shifts to **breadth-per-decision** (multiple lenses per move), comprehensive failure-mode coverage, and exhaustive path enumeration. This is explicit and intentional.

---

## Technique Selection

**Approach:** AI-Recommended Techniques

**Selected sequence:**

1. **Failure Analysis** (deep) — input: the L633→L692 forensic trace. Output: named failure modes + matched anti-pattern guards. These bind every subsequent next-move evaluation.
2. **Decision Tree Mapping** (structured) — input: L661-stable plan + Phase 1 guards. Output: enumerated next-action paths as a tree (pre-conditions, first-step, blast radius, what-would-invalidate-it, approval-required).
3. **Six Thinking Hats** (structured, applied to top 2-3 paths only) — input: top-ranked paths from Phase 2. Output: per-path multi-dimensional evaluation (White/Red/Black/Yellow/Green/Blue) suitable for committed decision or principled deferral.

---

## Technique Execution

### Phase 1 — Failure Analysis

**Input data:** L633→L692 trace from session jsonl `401fc7dc...`.

**Element 1 — Forensic timeline (already established, not re-derived):**
- L633 (user): Three pushbacks on integration-plan v1 — (a) Wave 2↔3 verification-conflation, (b) Wave 4 conflated concerns, (c) AsciiDoc mischaracterized as legacy. Plus "Justify."
- L637–L661 (Claude, GOOD): Three corrections correctly applied to plan. Sequencing now Wave 0→1→2(Evolutions)→3(FSM research)→4(decomposed)→5(hygiene).
- L670 (Claude, START OF DEGENERATION): Unprompted self-flag — "I don't know what success looks like ... reverting Wave 3's over-simplifications." User had not asked for this.
- L671–L684 (Claude, BAD): Sycophant edits — added meta-caveat block, rewrote Wave 3 as observer-only-research-no-feature-ship, stripped probability/impact ratings from risk register.
- L692 (user): Terminated. "Operating position is nearly worthless to me."
- L695+ (Claude): Reverted L671–L684. File returned to L661 state (current).

**Element 2 — Failure mode extraction:** to be filled in collaboratively. Each failure mode below names a specific behavior, the L-line evidence, the cause, and the guard that prevents recurrence.

[ideas captured below this line during execution]

