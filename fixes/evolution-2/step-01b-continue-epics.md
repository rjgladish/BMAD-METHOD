# Step 1b: Workflow Continuation Handler — Create Epics and Stories

---
name: 'step-01b-continue'
description: 'Handle workflow resumption when epics output already exists'

# Path Definitions
workflow_path: '{project-root}/_bmad/bmm/workflows/3-solutioning/create-epics-and-stories'

# File References
thisStepFile: './step-01b-continue.md'
nextStepFile: './step-02-design-epics.md'
workflowFile: '{workflow_path}/workflow.md'
outputFile: '{planning_artifacts}/epics.md'

# Template References
epicsTemplate: '{workflow_path}/templates/epics-template.md'
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

Handle workflow continuation by analyzing existing epics and stories work and guiding the user to resume at the appropriate step.

## CONTINUATION SEQUENCE:

### 1. Analyze Current Document State

Read the existing epics output document completely and analyze:

**Frontmatter Analysis:**

- `stepsCompleted`: What steps have been done
- `inputDocuments`: What documents were loaded
- `project_name`, `user_name`, `date`: Basic context
- `context`: How this document was created (e.g., brownfield synthesis vs. ceremony)

**Content Analysis:**

- How many epics exist in the document
- How many stories exist per epic
- Whether requirements (FRs, NFRs) have been extracted
- Whether the FR Coverage Map is populated
- Whether acceptance criteria exist on stories
- Any TODOs, placeholders, or incomplete sections remaining

### 2. Present Continuation Summary

Show the user their current progress:

"Welcome back {user_name}! I found your Epics and Stories work for {project_name}.

**Current Progress:**

- Steps completed: {stepsCompleted list}
- Creation method: {context field if present}
- Input documents used: {inputDocuments list}

**Document Status:**

- Epics found: {count of ## Epic N: sections}
- Stories found: {count of ### Story N.M: sections}
- Requirements extracted: {YES/NO — are FR/NFR sections populated?}
- Coverage map: {YES/NO — is requirements_coverage_map populated?}
- Acceptance criteria: {COMPLETE/PARTIAL/MISSING}

{if_incomplete_sections}
**Incomplete Areas:**

- {areas that appear incomplete or have placeholders}
{/if_incomplete_sections}

**What would you like to do?**
[R] Resume from where we left off (next incomplete step)
[C] Continue to next logical step
[O] Overview of all remaining steps
[X] Start over (will overwrite existing work)
"

### 3. Handle User Choice

#### If 'R' (Resume from where we left off):

Determine the next step based on document state:
- If NO requirements extracted → load `./step-01-validate-prerequisites.md` (skip section 0)
- If requirements extracted but NO epics_list → load `./step-02-design-epics.md`
- If epics_list exists but stories incomplete → load `./step-03-create-stories.md`
- If stories complete but NOT validated → load `./step-04-final-validation.md`

#### If 'C' (Continue to next logical step):

- Analyze document content to determine logical next step
- Review content quality and completeness of current step
- If content seems complete for current step, advance to next
- If content seems incomplete, suggest staying on current step

#### If 'O' (Overview of all remaining steps):

Present brief description of all steps with status:
1. **Validate Prerequisites** — Extract FRs, NFRs, additional requirements from input docs
2. **Design Epics** — Create and approve the epics_list organized by user value
3. **Create Stories** — Generate stories with acceptance criteria for each epic
4. **Final Validation** — Validate complete requirements coverage and readiness

Let user choose which step to work on.

#### If 'X' (Start over):

- Confirm: "This will delete all existing epics and stories. Are you sure? (y/n)"
- If confirmed: Delete existing document and load `./step-01-validate-prerequisites.md`
- If not confirmed: Return to continuation menu

### 4. Navigate to Selected Step

After user makes choice:

**Load the selected step file:**

- Update frontmatter `stepsCompleted` to reflect current navigation
- Execute the selected step file
- Let that step handle the detailed work

**State Preservation:**

- Maintain all existing content in the document
- Keep `stepsCompleted` accurate
- Do not lose any previously written epics or stories

### 5. Special Continuation Cases

#### If `stepsCompleted` contains 'synthesized-from-existing-artifacts':

- This means epics were created OUTSIDE the standard workflow (e.g., brownfield synthesis)
- The document may not follow the template exactly but contains valid work
- Present the content as-is and offer to continue from whatever step is logical
- Do NOT invalidate prior work just because it wasn't created through ceremony

#### If `stepsCompleted` is empty but document has content:

- This suggests an interrupted workflow
- Ask user: "I see the document has content but no steps are marked as complete. Should I analyze what's here and set the appropriate step status?"

#### If document appears corrupted or incomplete:

- Ask user: "The document seems incomplete. Would you like me to try to recover what's here, or would you prefer to start fresh?"

#### If all steps appear complete:

- Ask user: "The epics and stories look complete! Should I run final validation, or is there more you'd like to work on?"

## SUCCESS METRICS:

- Existing document state properly analyzed and understood
- User presented with clear continuation options
- User choice handled appropriately and transparently
- Workflow state preserved and updated correctly
- Navigation to appropriate step handled smoothly
- Previously created epics and stories NEVER lost during resumption

## FAILURE MODES:

- Not reading the complete existing document before making suggestions
- Losing track of what steps were actually completed
- Automatically proceeding without user confirmation of next steps
- Not checking for incomplete or placeholder content
- Losing existing document content during resumption
- CRITICAL: Ignoring 'synthesized-from-existing-artifacts' context and forcing fresh start
- CRITICAL: Proceeding without fully reading and understanding the selected step file

## NEXT STEP:

After user selects their continuation option, load the appropriate step file:

- `{workflow_path}/steps/step-01-validate-prerequisites.md`
- `{workflow_path}/steps/step-02-design-epics.md`
- `{workflow_path}/steps/step-03-create-stories.md`
- `{workflow_path}/steps/step-04-final-validation.md`

Remember: The goal is smooth, transparent resumption that respects the work already done while giving the user control over how to proceed.
