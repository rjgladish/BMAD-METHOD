# Patch: dev-story Mandatory Story Artifact Creation

**File:** `~/.claude/commands/bmad/dev-story.md`
**Section:** Part 10 — Commit and Update Status
**Applied:** 2026-04-02

## Before (defective)

```markdown
**Update story document** (if exists):
```markdown
## Progress Tracking

**Status History:**
- 2025-11-01: Created by Steve
...
```
```

The phrase "if exists" was an escape hatch that allowed agents to skip artifact
creation for stories with no pre-existing document.

## After (fixed)

```markdown
**Create or update story document** (MANDATORY — always required):

> **CRITICAL:** This step is NEVER optional. If a story document already exists, update it.
> If no story document exists yet, CREATE IT NOW at the project's artifact path before completing.
> The "if exists" framing in older versions of this workflow was a defect — it caused artifact files
> to be skipped for new stories. The rule is: every implemented story has an artifact file. No exceptions.
>
> **Artifact path by project convention:**
> - lkml: `_bmad-output/implementation-artifacts/{story-id}.md`
> - Default: `docs/stories/STORY-{ID}.md`
>
> Use an existing story artifact (e.g., the prior story in the same epic) as the template.
> Required sections: Story, Acceptance Criteria, Tasks (all checked), Dev Notes,
> Dev Agent Record (file list, completion notes), Completion Protocol checklist, Change Log.
```

## Why This Wording Works

1. **MANDATORY in the heading** — not buried in body text; visible at a glance
2. **CRITICAL callout block** — forces the agent to read before proceeding
3. **Explicitly names the defect** — "the 'if exists' framing was a defect" — agent cannot
   rationalize the old behavior as still valid
4. **Project-specific paths** — removes ambiguity about WHERE to write the file
5. **Template reference** — "use the prior story in the same epic" gives a concrete
   model to follow without requiring the agent to invent structure
6. **Required sections enumerated** — Story, AC, Tasks, Dev Notes, Dev Agent Record,
   Completion Protocol, Change Log — agent cannot produce a partial artifact and claim done

## Generalization

Any instruction of the form `do X (if Y)` where X is mandatory is a latent defect.
The conditional creates an escape hatch that agents will exploit — intentionally or not.

Fix pattern: Replace `do X (if Y)` with `do X (MANDATORY). If Y: update. If not Y: create.`
