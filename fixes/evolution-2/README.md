# Evolution-2: Workflow State Recovery Gate

**Date:** 2026-03-31
**Status:** APPLIED (P0-P2 HIGH)

## Problem

Step-file workflows assume greenfield execution. When re-invoked across sessions, they start from scratch, ignoring prior work stored in output files. The `stepsCompleted` frontmatter mechanism exists but is never checked on workflow re-entry.

## Fix Files

| File | Purpose |
|------|---------|
| `workflow-state-recovery-gate.md` | Root cause analysis and fix specification |
| `workflow-gate-patch-template.md` | Universal workflow.md patch template |
| `step-01b-continue-epics.md` | Continuation handler for create-epics-and-stories |
| `step-01b-continue-readiness.md` | Continuation handler for check-implementation-readiness |
| `step-01b-continue-project-context.md` | Continuation handler for generate-project-context |
| `yaml-workflow-state-recovery.md` | State recovery spec for YAML-based workflows |

## Applied Patches (in linkml project)

### P0 — create-epics-and-stories
- `workflow.md` — State Recovery Check added (section 2, step renumbered)
- `steps/step-01-validate-prerequisites.md` — Section 0 guard added
- `steps/step-01b-continue.md` — NEW FILE installed

### P1 — check-implementation-readiness
- `workflow.md` — State Recovery Check added (section 2, step renumbered)
- `steps/step-01-document-discovery.md` — Section 0 guard added
- `steps/step-01b-continue.md` — NEW FILE installed

### P1 — generate-project-context
- `workflow.md` — STATE RECOVERY CHECK section added before EXECUTION
- `steps/step-01-discover.md` — Section 0 guard added
- `steps/step-01b-continue.md` — NEW FILE installed

### P2-HIGH — sprint-planning
- `instructions.md` — State Recovery Check preamble added before Document Discovery

### P2-HIGH — create-story
- `instructions.xml` — `<step n="0">` state recovery gate added before step 1

### Deferred (P3)
- correct-course — date-stamped output rarely collides
- qa-generate-e2e-tests — usually regenerated fresh

### Not Applicable
- dev-story — output is code, not a recoverable document
- code-review — one-time analysis, no persistent output
- sprint-status — already operates on existing state
- retrospective — no defined persistent output file
