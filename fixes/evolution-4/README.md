# Evolution-4: dev-story Mandatory Story Artifact Creation

**Date:** 2026-04-02
**Status:** APPLIED (dev-story command patched)

## Problem

The `dev-story` workflow (Part 10, Commit and Update Status) contained the
instruction:

> **Update story document** (if exists):

The phrase "if exists" was interpreted as permission to skip creating the story
artifact file when no prior document existed for the story being implemented.
This produced invisible stories — implemented, tested, sprint-status updated,
journal written — but with no artifact file in the project's
`_bmad-output/implementation-artifacts/` directory.

**Impact:**
1. Dashboard (sprint board, artifact index) has no visibility into what was built
2. Acceptance criteria, task checklist, and completion notes are unrecorded
3. Future agents have no story file to reference for context
4. The Completion Protocol checklist (stage → commit → push) is left unchecked
5. The pattern established by E1-S1 through E1-S4 (each story has an artifact) is violated silently

**Root cause:** Ambiguous conditional on a mandatory action. "If exists" was
intended as "update if there's already a file, create if not" but was misread
as "only act if the file already exists."

**Discovered:** SRCH-1.5 (lkml project) was implemented without creating
`_bmad-output/implementation-artifacts/srch-1-5-validate-spec-and-exemplar.md`.
The artifact was created retroactively.

## Fix

Changed `**Update story document** (if exists):` to
`**Create or update story document** (MANDATORY — always required):` with an
explicit callout explaining:

- This step is NEVER optional
- If no document exists: CREATE IT NOW
- The "if exists" framing was a defect
- Every implemented story has an artifact file — no exceptions
- Use the prior story in the same epic as the template
- Lists required sections: Story, AC, Tasks, Dev Notes, Dev Agent Record, Completion Protocol, Change Log
- Notes project-specific artifact paths (lkml: `_bmad-output/implementation-artifacts/`)

## Fix Files

| File | Purpose |
|------|---------|
| `README.md` | This file — root cause analysis |
| `dev-story-artifact-creation-patch.md` | Exact before/after for the dev-story command |

## Applied Patches

### P0 — dev-story command (APPLIED 2026-04-02)

- `~/.claude/commands/bmad/dev-story.md` — Part 10 "Update story document (if exists)"
  → "Create or update story document (MANDATORY — always required)" with explicit
  no-exceptions callout, project-specific path guidance, and required section list.

## Related Evolutions

| Evolution | Problem | Status |
|-----------|---------|--------|
| Evolution-1 | (see evolution-1/README.md) | Applied |
| Evolution-2 | Workflow state recovery gate — step-file workflows restart from scratch across sessions | Applied |
| Evolution-3 | Workflow status registry write-back — `bmm-workflow-status.yaml` never updated on completion | Spec complete, patches pending |
| Evolution-4 | dev-story artifact creation mandatory — "if exists" escape hatch removed | Applied |

## Testing

After applying this patch, run a dev-story that has no prior artifact file:

1. Pick a `backlog` or `ready-for-dev` story with no pre-existing `.md` in the artifact dir
2. Run `/bmad:dev-story {story-id}`
3. After implementation: verify artifact file exists in `_bmad-output/implementation-artifacts/`
4. Verify Completion Protocol checklist in the artifact is fully checked
