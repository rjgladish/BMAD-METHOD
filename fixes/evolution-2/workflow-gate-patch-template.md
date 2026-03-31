# Universal Workflow Gate Patch — Template

**Purpose:** This patch adds a State Recovery Check between Configuration Loading and First Step EXECUTION in every step-file workflow's `workflow.md`.

**How to apply:** Insert the section below between the existing "Configuration Loading" (or "Module Configuration Loading") section and the "First Step EXECUTION" (or "EXECUTION") section. Renumber the First Step section accordingly.

---

## Patch Content

Insert this section after Configuration Loading:

```markdown
### 2. State Recovery Check

Before executing any step, check if the output document already exists:

1. Search for the workflow's output file (see step file frontmatter for `outputFile` path)
2. **IF** output file exists **AND** has frontmatter with `stepsCompleted` (non-empty array):
   - Read the output file completely
   - Route to `./steps/step-01b-continue.md` instead of step-01
   - Do NOT proceed with step-01 initialization
3. **IF** output file exists **BUT** has NO `stepsCompleted` or empty array:
   - Treat as interrupted workflow — content exists but state is unknown
   - Route to `./steps/step-01b-continue.md` with interrupted state
4. **IF** output file does NOT exist:
   - Proceed normally to step-01

**Output file discovery patterns** (check in order):
- Exact path from step frontmatter `outputFile` field
- Glob `{planning_artifacts}/*<workflow-keyword>*.md`
- Glob `{output_folder}/*<workflow-keyword>*.md`
```

Then renumber "First Step EXECUTION" from 2 → 3.

---

## Workflow-Specific Output File Patterns

| Workflow | outputFile | Discovery Glob |
|----------|-----------|----------------|
| create-epics-and-stories | `{planning_artifacts}/epics.md` | `{planning_artifacts}/*epic*.md` |
| check-implementation-readiness | `{planning_artifacts}/readiness-report.md` | `{planning_artifacts}/*readiness*.md` |
| generate-project-context | `{output_folder}/project-context.md` | `**/project-context.md` |
