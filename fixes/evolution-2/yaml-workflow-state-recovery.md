# Evolution-2 Fix: YAML Workflow State Recovery

**Problem:** YAML-based workflows (Phase 4 implementation) use `instructions.xml/md` with `<step>` elements instead of step-file architecture. They lack the `stepsCompleted` frontmatter mechanism entirely, so state recovery requires a different approach.

**Approach:** Add a `<step n="0">` state recovery gate to the beginning of instructions files that produce persistent output. This step checks for existing output before proceeding.

---

## Applicability Assessment

| Workflow | Output File | State Recovery Applicable? | Priority |
|----------|-----------|--------------------------|----------|
| sprint-planning | `sprint-status.yaml` | **YES** — recreating destroys story status | P2-HIGH |
| create-story | `{story_key}.md` | **YES** — recreating loses enriched context | P2-HIGH |
| dev-story | code changes | NO — output is code, not document | N/A |
| code-review | ephemeral | NO — review is one-time analysis | N/A |
| correct-course | `sprint-change-proposal-{date}.md` | LOW — date-stamped, rarely collides | P3 |
| qa-generate-e2e-tests | `test-summary.md` | LOW — usually regenerated fresh | P3 |
| sprint-status | reads existing YAML | NO — already operates on existing state | N/A |
| retrospective | no output file defined | NO — ephemeral analysis | N/A |

---

## Fix: sprint-planning instructions.md

Add before existing `<step n="1">`:

```xml
<step n="0" goal="Check for existing sprint status">
  <action>Check if {status_file} (sprint-status.yaml) already exists</action>
  <check if="sprint-status.yaml exists and has content">
    <action>Read the existing sprint-status.yaml completely</action>
    <action>Count stories by status: backlog, in-progress, done, blocked</action>
    <output>
      **Existing Sprint Status Found!**

      Status file: {status_file}
      Stories tracked: {total_count}
      - Backlog: {backlog_count}
      - In Progress: {in_progress_count}
      - Done: {done_count}
      - Blocked: {blocked_count}

      **What would you like to do?**
      [U] Update — Add new stories from epics while preserving existing status
      [R] Regenerate — Start fresh (WARNING: loses all story status tracking)
      [V] View — Show current status details, then decide
    </output>
    <ask>Choose [U]pdate, [R]egenerate, or [V]iew:</ask>

    <check if="user chooses U">
      <action>Load existing sprint-status.yaml as baseline</action>
      <action>Proceed to step 1 with merge mode — only ADD stories not already tracked</action>
      <action>NEVER overwrite status of existing stories</action>
    </check>

    <check if="user chooses R">
      <ask>Confirm: This will delete all existing story status. Are you sure? (y/n)</ask>
      <check if="confirmed">
        <action>Proceed to step 1 in fresh mode</action>
      </check>
      <check if="not confirmed">
        <action>Return to state recovery menu</action>
      </check>
    </check>

    <check if="user chooses V">
      <action>Display full sprint-status.yaml contents formatted as readable table</action>
      <action>Return to state recovery menu after display</action>
    </check>
  </check>
  <check if="sprint-status.yaml does NOT exist">
    <action>Proceed to step 1 in fresh mode</action>
  </check>
</step>
```

---

## Fix: create-story instructions.xml

Add before existing `<step n="1">`:

```xml
<step n="0" goal="Check for existing story file">
  <action>Determine target story from user input or sprint-status auto-discovery</action>
  <action>Check if the target story file already exists at {default_output_file}</action>
  <check if="story file exists and has content">
    <action>Read the existing story file completely including frontmatter</action>
    <output>
      **Existing Story File Found!**

      File: {story_file_path}
      Story: {story_title}
      Status: {frontmatter status if present}

      The story file already exists with content. This may be from a prior session.

      **What would you like to do?**
      [C] Continue — Resume enriching this story from where it left off
      [R] Recreate — Start fresh (WARNING: loses existing story context)
      [V] View — Show current story contents, then decide
    </output>
    <ask>Choose [C]ontinue, [R]ecreate, or [V]iew:</ask>

    <check if="user chooses C">
      <action>Load existing story content as baseline</action>
      <action>Skip to enrichment steps (step 2a+), preserving existing content</action>
    </check>

    <check if="user chooses R">
      <ask>Confirm: This will overwrite the existing story. Are you sure? (y/n)</ask>
      <check if="confirmed">
        <action>Proceed to step 1 in fresh mode</action>
      </check>
    </check>

    <check if="user chooses V">
      <action>Display full story file contents</action>
      <action>Return to state recovery menu</action>
    </check>
  </check>
  <check if="story file does NOT exist">
    <action>Proceed to step 1 in fresh mode</action>
  </check>
</step>
```

---

## Deferred (P3)

- **correct-course**: Date-stamped output rarely collides. Add state recovery when workflow is frequently used.
- **qa-generate-e2e-tests**: Usually regenerated. Add when test generation becomes incremental.

## Not Applicable

- **dev-story**: Output is code, not a recoverable document.
- **code-review**: One-time analysis, no persistent output.
- **sprint-status**: Already reads existing state by design.
- **retrospective**: Ephemeral analysis with no defined output file.
