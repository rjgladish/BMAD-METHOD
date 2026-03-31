# Step 1b: Workflow Continuation Handler — Check Implementation Readiness

---
name: 'step-01b-continue'
description: 'Handle workflow resumption when readiness report already exists'

# Path Definitions
workflow_path: '{project-root}/_bmad/bmm/workflows/3-solutioning/check-implementation-readiness'

# File References
thisStepFile: './step-01b-continue.md'
nextStepFile: './step-02-prd-analysis.md'
workflowFile: '{workflow_path}/workflow.md'
outputFile: '{planning_artifacts}/readiness-report.md'
---

## MANDATORY EXECUTION RULES (READ FIRST):

- NEVER generate content without user input
- CRITICAL: ALWAYS read the complete step file before taking any action
- CRITICAL: When loading next step with 'C', ensure the entire file is read and understood before proceeding
- YOU ARE A FACILITATOR, not a content generator
- FOCUS on understanding current state and getting user confirmation
- HANDLE workflow resumption smoothly and transparently
- YOU MUST ALWAYS SPEAK OUTPUT In your Agent communication style with the config `{communication_language}`

## EXECUTION PROTOCOLS:

- Show your analysis before taking any action
- Read existing document completely to understand current state
- Update frontmatter to reflect continuation
- FORBIDDEN to proceed to next step without user confirmation

## CONTEXT BOUNDARIES:

- Existing document and frontmatter are available
- Input documents already loaded should be in frontmatter `inputDocuments`
- Steps already completed are in `stepsCompleted` array
- Focus on understanding where we left off

## YOUR TASK:

Handle workflow continuation by analyzing existing readiness report and guiding the user to resume at the appropriate step.

## CONTINUATION SEQUENCE:

### 1. Analyze Current Document State

Read the existing readiness report completely and analyze:

**Frontmatter Analysis:**

- `stepsCompleted`: What steps have been done
- `inputDocuments`: What documents were loaded
- `project_name`, `user_name`, `date`: Basic context

**Content Analysis:**

- Which validation sections are complete (document discovery, PRD analysis, epic coverage, UX alignment, epic quality, final assessment)
- What findings have been recorded
- Any TODOs or placeholders remaining
- Overall readiness verdict if present

### 2. Present Continuation Summary

"Welcome back {user_name}! I found your Implementation Readiness Report for {project_name}.

**Current Progress:**

- Steps completed: {stepsCompleted list}
- Input documents loaded: {inputDocuments count}

**Validation Status:**

- Document Discovery: {DONE/PENDING}
- PRD Analysis: {DONE/PENDING}
- Epic Coverage Validation: {DONE/PENDING}
- UX Alignment: {DONE/PENDING/N/A}
- Epic Quality Review: {DONE/PENDING}
- Final Assessment: {DONE/PENDING}

{if_incomplete_sections}
**Incomplete Areas:**
- {areas that appear incomplete or have placeholders}
{/if_incomplete_sections}

**What would you like to do?**
[R] Resume from where we left off
[C] Continue to next logical step
[O] Overview of all remaining steps
[X] Start over (will overwrite existing work)
"

### 3. Handle User Choice

#### If 'R' (Resume):

Determine next step based on `stepsCompleted`:
- Missing step 1 → `./step-01-document-discovery.md` (skip section 0)
- Missing step 2 → `./step-02-prd-analysis.md`
- Missing step 3 → `./step-03-epic-coverage-validation.md`
- Missing step 4 → `./step-04-ux-alignment.md`
- Missing step 5 → `./step-05-epic-quality-review.md`
- Missing step 6 → `./step-06-final-assessment.md`

#### If 'C' (Continue to next logical step):

- Analyze content to determine logical next step
- If current step seems complete, advance
- If incomplete, suggest staying

#### If 'O' (Overview):

Present all steps with status:
1. **Document Discovery** — Find and validate all planning documents
2. **PRD Analysis** — Validate requirements completeness and traceability
3. **Epic Coverage Validation** — Verify all requirements covered by epics
4. **UX Alignment** — Check UX design alignment with epics (if applicable)
5. **Epic Quality Review** — Review story quality, acceptance criteria, dependencies
6. **Final Assessment** — Overall readiness verdict and recommendations

#### If 'X' (Start over):

- Confirm: "This will delete the existing readiness report. Are you sure? (y/n)"
- If confirmed: Delete and load `./step-01-document-discovery.md`
- If not confirmed: Return to menu

### 4. Navigate to Selected Step

- Update frontmatter to reflect current navigation
- Execute the selected step file
- Maintain all existing content

### 5. Special Continuation Cases

#### If `stepsCompleted` is empty but document has content:

- Ask: "Document has content but no steps marked complete. Analyze and set status?"

#### If document appears complete:

- Ask: "The readiness report looks complete! Run final assessment again, or done?"

## SUCCESS METRICS:

- Existing report state properly analyzed
- User presented with clear options
- Workflow state preserved correctly
- Previously completed validations NEVER lost

## FAILURE MODES:

- Not reading complete existing document
- Losing completed validation findings
- Automatically proceeding without confirmation
- Re-running validations that are already complete

## VALID STEP FILES:

- `{workflow_path}/steps/step-01-document-discovery.md`
- `{workflow_path}/steps/step-02-prd-analysis.md`
- `{workflow_path}/steps/step-03-epic-coverage-validation.md`
- `{workflow_path}/steps/step-04-ux-alignment.md`
- `{workflow_path}/steps/step-05-epic-quality-review.md`
- `{workflow_path}/steps/step-06-final-assessment.md`
