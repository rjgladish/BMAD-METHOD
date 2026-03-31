# Evolution-2 Fix: Workflow State Recovery Gate

**Problem:** Step-file workflows assume greenfield execution. When invoked across multiple sessions (common for multi-week projects), the workflow starts from scratch — re-discovering documents, re-extracting requirements — ignoring weeks of prior work stored in the output file.

**Root Cause:** The INITIALIZATION SEQUENCE in `workflow.md` has no state recovery step. It loads config, then immediately routes to `step-01-*`. The `stepsCompleted` frontmatter mechanism exists for tracking state but is never checked on workflow re-entry.

**Impact:** All step-file workflows that produce output documents are affected. Some workflows (architecture, product-brief, PRD, UX, brainstorming) already have `step-01b-continue.md` handlers. Others (create-epics-and-stories, check-implementation-readiness, generate-project-context, and most implementation workflows) do not.

**Affected Workflows (missing step-01b-continue.md):**

| Workflow | Phase | Has step-01b? | Produces Output? |
|----------|-------|---------------|-------------------|
| bmad-create-epics-and-stories | 3-solutioning | NO | YES (epics.md) |
| bmad-check-implementation-readiness | 3-solutioning | NO | YES (readiness-report.md) |
| bmad-generate-project-context | 3-solutioning | NO | YES (project-context.md) |
| bmad-create-story | 4-implementation | NO | YES (story files) |
| bmad-dev-story | 4-implementation | NO | YES (story files) |
| bmad-sprint-planning | 4-implementation | NO | YES (sprint plan) |
| bmad-code-review | 4-implementation | NO | Possibly |
| bmad-correct-course | 4-implementation | NO | YES (change proposal) |
| bmad-qa-generate-e2e-tests | 4-implementation | NO | YES (test specs) |
| bmad-sprint-status | 4-implementation | NO | YES (status report) |
| bmad-retrospective | 4-implementation | NO | YES (retro report) |

**Workflows with existing step-01b (no fix needed):**

| Workflow | Phase |
|----------|-------|
| bmad-create-product-brief | 1-analysis |
| bmad-create-prd | 2-plan-workflows |
| bmad-create-ux-design | 2-plan-workflows |
| bmad-create-architecture | 3-solutioning |
| bmad-brainstorming | core-skills |

---

## Fix 1: Universal State Recovery Gate in workflow.md

**Location:** Every `workflow.md` that uses step-file architecture

**Change:** Add a new section between "Configuration Loading" and "First Step EXECUTION" in the INITIALIZATION SEQUENCE:

```markdown
### 2. State Recovery Check

Before executing any step, check if the output document already exists:

1. Search for {outputFile} (resolved from step file frontmatter or config)
2. IF output file exists AND has frontmatter with `stepsCompleted`:
   - Read the output file completely
   - Route to `./steps/step-01b-continue.md` instead of step-01
   - Do NOT proceed with step-01 initialization
3. IF output file exists but has NO `stepsCompleted` or empty array:
   - Treat as interrupted workflow
   - Route to `./steps/step-01b-continue.md` with interrupted state
4. IF output file does NOT exist:
   - Proceed normally to `./steps/step-01-*.md`

### 3. First Step EXECUTION

(renumbered from 2)

Read fully and follow: `./steps/step-01-*.md` to begin the workflow.
```

**Rationale:** This is a universal preamble that works for ALL step-file workflows. The output file's frontmatter IS the state — if it exists, the workflow has prior state to recover.

---

## Fix 2: step-01b-continue.md Template

**Location:** Each affected workflow's `steps/` directory

Each workflow needs its own `step-01b-continue.md` adapted to its output structure, but they all follow the same pattern established by `bmad-create-architecture/steps/step-01b-continue.md`.

See companion file: `step-01b-continue-epics.md` for the create-epics-and-stories specific implementation.

---

## Fix 3: step-01 Init/Validate Must Check for Existing Output

**Location:** Each affected workflow's `step-01-*.md`

The existing `step-01-init.md` pattern in architecture is correct — it checks for existing output FIRST, then routes to step-01b if found. Workflows that have `step-01-validate-prerequisites.md` or similar init steps need the same guard added at the top of their execution sequence.

**Change to step-01 files (add before section 1):**

```markdown
### 0. Check for Existing Workflow State

Before beginning initialization:

1. Check if {outputFile} already exists
2. If it exists and has `stepsCompleted` in frontmatter:
   - **STOP here** and load `./step-01b-continue.md` immediately
   - Do not proceed with any initialization tasks
   - Let step-01b handle the continuation logic
3. If no document exists, proceed with fresh initialization below
```

---

## Implementation Priority

1. **P0:** Fix `create-epics-and-stories` (triggered this discovery, immediate user pain)
2. **P1:** Fix all Phase 3 solutioning workflows (long-running, multi-session)
3. **P2:** Fix Phase 4 implementation workflows (shorter but still benefit)
4. **P3:** Update workflow.md template in BMAD core for new workflows
