# Evolution-3: Workflow Status Registry Write-Back

**Date:** 2026-03-31
**Status:** SPEC COMPLETE, PATCHES PENDING
**Depends on:** evolution-2 (workflow state recovery gate)

## Problem

Individual BMM workflows track their own completion state in output file
frontmatter (`stepsCompleted` array), but **never write back** to the central
`bmm-workflow-status.yaml` registry. This means:

1. `/bmad:workflow-status` dashboard cannot detect completed workflows
2. Phase progression logic in `helpers.md#Determine-Next-Workflow` fails
3. The `workflow-init` command creates `bmm-workflow-status.yaml` but nothing
   ever updates it — all entries stay at their initial status forever
4. Brownfield projects that skip `workflow-init` never get the file at all

The PRD and Architecture appear to work in the dashboard only because some
commands fall back to glob-matching `*prd*.md` / `*architecture*.md` in the
output folder. Epics, research, UX-design, and gate-check have no such fallback.

## Root Cause

The framework has two independent state systems that don't talk to each other:

| System | Where | Written by | Read by |
|--------|-------|-----------|---------|
| Per-workflow frontmatter | `{planning_artifacts}/*.md` | Each workflow's final step | That workflow's `step-01b-continue.md` |
| Central status registry | `bmm-workflow-status.yaml` | `workflow-init` (once) | `/bmad:workflow-status` dashboard |

The bridge between them — **writing to the registry on workflow completion** —
was never implemented.

## Fix Specification

### Fix A: Workflow completion write-back

Every workflow's final step (the "complete" step) must update
`bmm-workflow-status.yaml` after saving the output document:

```yaml
# Pseudo-instruction to add to each workflow's final step:
#
# WORKFLOW STATUS UPDATE:
# 1. Check if {bmm.workflow_status_file} exists (from project bmad/config.yaml)
# 2. If it exists:
#    - Find the entry matching this workflow's name
#    - Update status to the output file path
#    - Update last_updated to current date
# 3. If it does NOT exist:
#    - Create it from template with this single entry populated
#    - Log: "Created bmm-workflow-status.yaml — run /bmad:workflow-status for full state"
```

### Fix B: Auto-create on first completion (missing file resilience)

If `bmm-workflow-status.yaml` doesn't exist when a workflow completes, create
it from the template (`~/.claude/config/bmad/templates/bmm-workflow-status.template.yaml`)
with the completing workflow's entry populated. This handles brownfield projects
that never ran `workflow-init`.

### Fix C: Missing project config resilience

If `bmad/config.yaml` doesn't exist, the dashboard exits with "not initialized."
The dashboard should fall back to `_bmad/bmm/config.yaml` (module config) when
the slim-install project config is absent. Many projects use the full `_bmad/`
install without the slim `bmad/` overlay.

## Affected Workflows (completion steps that need write-back)

| Workflow | Final Step | Has status update? |
|----------|-----------|-------------------|
| create-product-brief | step-06-complete.md | Mentions it, **but target file may not exist** |
| create-prd | step-12-complete.md | Mentions it, conditional on "if exists" |
| create-ux-design | step-14-complete.md | YES — has explicit update block |
| create-architecture | step-08-complete.md | Mentions "update workflow status" in failures |
| create-epics-and-stories | step-04-final-validation.md | **NO — completely missing** |
| check-implementation-readiness | (no completion step) | **NO — no write-back at all** |
| sprint-planning | workflow.yaml | **NO** |
| research (all 3) | step-06-*.md | Mentions it in passing |

## Patch Template

Add to every final/completion step, after the document save:

```markdown
### Workflow Status Registry Update

After saving the output document:

1. Resolve `{workflow_status_file}` from project config (`bmad/config.yaml` → `bmm.workflow_status_file`)
   - Fallback: `{output_folder}/bmm-workflow-status.yaml`
   - Fallback: `_bmad-output/bmm-workflow-status.yaml`
2. **IF** status file exists:
   - Find entry where `name` matches this workflow name
   - Set `status` to the output file path (relative to project root)
   - Set `last_updated` to current date (YYYY-MM-DD)
3. **IF** status file does NOT exist:
   - Load template from `~/.claude/config/bmad/templates/bmm-workflow-status.template.yaml`
   - Substitute project variables from config
   - Set this workflow's entry to the output file path
   - Write to the resolved path
   - Inform user: "Created workflow status registry at {path}"
```

### Fix D: Orchestrator-level registry audit (defense in depth)

Self-reporting by workflows is necessary but not sufficient. The same pattern
(each workflow responsible for its own write-back) already failed once — a
workflow that forgets to write back silently breaks the dashboard.

The orchestrator layer must independently verify registry consistency:

**D1: `/bmad:workflow-status` dashboard — reconciliation on read**

When the dashboard loads, BEFORE displaying status, it should:

1. Read `bmm-workflow-status.yaml` (the registry)
2. Glob for known output artifacts in `{planning_artifacts}/`:
   - `*prd*.md` → prd
   - `*architecture*.md` → architecture
   - `*epic*.md` → create-epics-and-stories
   - `*ux*.md` → create-ux-design
   - `*readiness*.md` or `*gate*.md` → solutioning-gate-check
   - `*product-brief*.md` → product-brief
3. For each artifact found on disk but NOT marked complete in registry:
   - Display warning: `⚠ {workflow} artifact exists ({file}) but registry shows "{status}"`
   - Offer: `[F] Fix registry to match disk state`
4. For each entry marked complete in registry but artifact missing on disk:
   - Display warning: `⚠ {workflow} marked complete but {file} not found`

This makes the dashboard self-healing — even if a workflow forgets to write
back, the next `/bmad:workflow-status` call catches and repairs the drift.

**D2: bmad-master menu — post-workflow completion hook**

When bmad-master hands off to a workflow and that workflow completes (returns
control to bmad-master), bmad-master should:

1. Check if the workflow produced an output file
2. Check if `bmm-workflow-status.yaml` was updated
3. If NOT updated: perform the write-back on behalf of the workflow
4. Log: "📋 Registry updated for {workflow} → {output_file}"

This is the supervisor pattern — the orchestrator doesn't trust subordinates
to self-report; it verifies and patches.

**D3: `bmm-workflow-status.template.yaml` — add missing entries**

The template is missing `create-epics-and-stories` entirely. Add to Phase 3:

```yaml
  - name: create-epics-and-stories
    phase: 3
    status: "required"
    description: "Epic and story breakdown"
```

Without this entry, even correct write-back code has no target row to update.

## Applied Patches (in linkml project)

### Retroactive state files created

- `bmad/config.yaml` — Slim-install project config (was missing entirely)
- `_bmad-output/bmm-workflow-status.yaml` — Central registry populated from existing artifacts

### Framework patches (pending upstream push)

- [ ] `step-04-final-validation.md` (create-epics-and-stories) — Add write-back
- [ ] `step-08-complete.md` (create-architecture) — Fix write-back to be explicit
- [ ] `step-06-complete.md` (create-product-brief) — Make write-back unconditional
- [ ] `step-12-complete.md` (create-prd) — Make write-back unconditional
- [ ] `workflow-status.md` (dashboard command) — Add fallback to `_bmad/bmm/config.yaml`
- [ ] `workflow-status.md` (dashboard command) — Add reconciliation-on-read (Fix D1)
- [ ] `helpers.md` — Add `_bmad/bmm/config.yaml` fallback in Load Project Config
- [ ] `bmm-workflow-status.template.yaml` — Add `create-epics-and-stories` entry (Fix D3)
- [ ] bmad-master agent — Add post-workflow completion verification hook (Fix D2)

## Testing

After applying patches:
1. Run `/bmad:workflow-status` — should show all completed workflows with file paths
2. Run `/bmad-bmm-create-epics-and-stories` — should detect existing state via both
   frontmatter AND registry
3. Delete `bmm-workflow-status.yaml`, complete any workflow — file should be auto-created
4. Remove a registry entry but leave the artifact on disk — dashboard should detect
   and offer to fix the drift (Fix D1)
5. Complete a workflow with write-back intentionally broken — bmad-master should
   catch and repair on return (Fix D2)
