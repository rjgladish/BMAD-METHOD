# Workflow Completion Write-Back Patch Template

**Purpose:** Universal patch block to append to every workflow's final/completion step.
Insert this AFTER the "Save output document" section and BEFORE the "Present Final Menu" section.

---

## PATCH BLOCK — Copy into each completion step

```markdown
### Workflow Status Registry Update

After saving the output document, update the central workflow status registry:

1. **Resolve status file path:**
   - Check `bmad/config.yaml` → `bmm.workflow_status_file`
   - Fallback: `{output_folder}/bmm-workflow-status.yaml`
   - Fallback: `_bmad-output/bmm-workflow-status.yaml`

2. **IF** status file exists:
   - Read the YAML file
   - Find the entry in `workflow_status` array where `name` matches `{workflow_name}`
   - Update `status` field to the relative path of the saved output file
   - Update root-level `last_updated` to current date (YYYY-MM-DD format)
   - Save the modified YAML back

3. **IF** status file does NOT exist:
   - Load template: `~/.claude/config/bmad/templates/bmm-workflow-status.template.yaml`
   - Substitute variables from project config:
     - `{{PROJECT_NAME}}` → `{project_name}`
     - `{{PROJECT_TYPE}}` → from `bmad/config.yaml` or default "library"
     - `{{PROJECT_LEVEL}}` → from `bmad/config.yaml` or default 2
     - `{{TIMESTAMP}}` → current date
     - `{{PRD_STATUS}}` → "required" (level 2+) or "recommended" (level 0-1)
     - `{{TECH_SPEC_STATUS}}` → "required" (level 0-1) or "optional" (level 2+)
     - `{{ARCHITECTURE_STATUS}}` → "required" (level 2+) or "optional" (level 0-1)
   - Set this workflow's entry `status` to the output file path
   - Write to the resolved path
   - Inform user: "📋 Created workflow status registry at {path}"

4. **IF** project config (`bmad/config.yaml`) does not exist:
   - Use `_bmad/bmm/config.yaml` as fallback for `project_name` and `output_folder`
   - Default `workflow_status_file` to `{output_folder}/bmm-workflow-status.yaml`
   - Log: "ℹ No bmad/config.yaml found — using _bmad/bmm/config.yaml defaults"
```

---

## Workflow-to-Name Mapping

Use these exact `name` values when finding the matching entry in the registry:

| Workflow | Registry `name` |
|----------|----------------|
| create-product-brief | `product-brief` |
| create-prd | `prd` |
| create-ux-design | `create-ux-design` |
| create-architecture | `architecture` |
| create-epics-and-stories | `create-epics-and-stories` |
| check-implementation-readiness | `solutioning-gate-check` |
| sprint-planning | `sprint-planning` |
| research (domain) | `research` |
| research (market) | `research` |
| research (technical) | `research` |

**NOTE:** The `create-epics-and-stories` entry does NOT exist in the current
`bmm-workflow-status.template.yaml`. The template needs an additional entry
added to the Phase 3 section:

```yaml
  - name: create-epics-and-stories
    phase: 3
    status: "required"
    description: "Epic and story breakdown"
```

This is a **template defect** — the template tracks architecture and
solutioning-gate-check but omits the epics workflow entirely.

---

## Dashboard Fallback Patch (workflow-status.md)

In `~/.claude/commands/bmad/workflow-status.md`, Step 1 should be patched:

**Current (Step 1):**
```
1. Check if `bmad/config.yaml` exists
2. If NOT exists: → Exit with "BMAD not initialized"
```

**Patched (Step 1):**
```
1. Check if `bmad/config.yaml` exists
2. If NOT exists:
   a. Check if `_bmad/bmm/config.yaml` exists (full module install)
   b. If THAT exists: use it as project config, derive output_folder, continue
   c. If NEITHER exists: → Exit with "BMAD not initialized"
```

---

## Helpers Fallback Patch (helpers.md)

In `~/.claude/config/bmad/helpers.md`, section "Load Project Config":

**Current:**
```
Path: {project-root}/bmad/config.yaml
```

**Patched:**
```
Path (primary):  {project-root}/bmad/config.yaml
Path (fallback): {project-root}/_bmad/bmm/config.yaml

Using Read tool:
1. Try bmad/config.yaml first
2. If not found, try _bmad/bmm/config.yaml
3. If neither found, report "BMAD not initialized"
```
