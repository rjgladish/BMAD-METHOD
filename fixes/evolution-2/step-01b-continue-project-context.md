# Step 1b: Workflow Continuation Handler — Generate Project Context

---
name: 'step-01b-continue'
description: 'Handle workflow resumption when project-context.md already exists'

# Path Definitions
workflow_path: '{project-root}/_bmad/bmm/workflows/generate-project-context'

# File References
thisStepFile: './step-01b-continue.md'
nextStepFile: './step-02-generate.md'
workflowFile: '{workflow_path}/workflow.md'
outputFile: '{output_folder}/project-context.md'
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

## YOUR TASK:

Handle workflow continuation by analyzing existing project-context.md and guiding the user to resume at the appropriate step.

## CONTINUATION SEQUENCE:

### 1. Analyze Current Document State

Read the existing project-context.md completely and analyze:

**Frontmatter Analysis:**

- `stepsCompleted`: What steps have been done
- `inputDocuments`: What documents were scanned
- `project_name`, `user_name`, `date`: Basic context

**Content Analysis:**

- What sections exist (rules, patterns, guidelines, constraints)
- Whether content appears complete or has placeholders
- Quality of existing AI rules and patterns
- Any TODOs or areas needing expansion

### 2. Present Continuation Summary

"Welcome back {user_name}! I found your Project Context file for {project_name}.

**Current Progress:**

- Steps completed: {stepsCompleted list}
- Documents scanned: {inputDocuments count}

**Content Status:**

- Sections found: {list of H2/H3 sections}
- Content quality: {assessment — minimal/partial/comprehensive}

{if_incomplete_sections}
**Incomplete Areas:**
- {areas with placeholders or minimal content}
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
- Missing step 1 → `./step-01-discover.md` (skip section 0)
- Missing step 2 → `./step-02-generate.md`
- Missing step 3 → `./step-03-complete.md`

#### If 'C' (Continue to next logical step):

- Analyze content to determine logical next step
- If current step seems complete, advance
- If incomplete, suggest staying

#### If 'O' (Overview):

Present all steps with status:
1. **Discover** — Scan codebase and planning docs for patterns and rules
2. **Generate** — Create optimized AI rules and patterns content
3. **Complete** — Finalize and validate the project-context.md

#### If 'X' (Start over):

- Confirm: "This will delete the existing project context. Are you sure? (y/n)"
- If confirmed: Delete and load `./step-01-discover.md`
- If not confirmed: Return to menu

### 4. Navigate to Selected Step

- Update frontmatter to reflect current navigation
- Execute the selected step file
- Maintain all existing content

### 5. Special Continuation Cases

#### If project-context.md exists but was manually created:

- Present content and offer to enhance rather than replace
- Ask: "This appears to be manually maintained. Should I enhance it or create a workflow-generated version alongside it?"

#### If document appears complete:

- Ask: "The project context looks complete! Want to review and refine, or mark as done?"

## SUCCESS METRICS:

- Existing context properly analyzed
- User presented with clear options
- Existing rules and patterns NEVER lost
- Workflow state preserved correctly

## FAILURE MODES:

- Not reading complete existing document
- Overwriting manually maintained project-context.md
- Losing existing AI rules during resumption
- Automatically proceeding without confirmation

## VALID STEP FILES:

- `{workflow_path}/steps/step-01-discover.md`
- `{workflow_path}/steps/step-02-generate.md`
- `{workflow_path}/steps/step-03-complete.md`
