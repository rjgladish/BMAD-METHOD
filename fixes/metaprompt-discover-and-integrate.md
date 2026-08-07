# Metaprompt — BMAD Cross-Project Evolution Discovery and Integration (v2)

**Mission.** Surface every BMAD fix, ad-hoc workaround, and defect that has accumulated across the user's projects, then sequence them into a coherent integration plan against this repo's `enhance/better-adr-capture` branch.

**Operating context.**
- Working tree: `~/repos/harness/skills/BMAD-METHOD.git` (the BMAD-METHOD substrate itself).
- This repo is **excluded** from "BMAD-using project" analysis. It is the source of truth, not a consumer.
- User runs many concurrent worktrees and subagent copies. Naive filesystem walks will read the same files dozens of times. Every search step below specifies how to dedup before reading.
- Discovery is **read-only** outside this repo. Edits land only on `enhance/better-adr-capture` here.
- House rules: token discipline, no fake BMAD execution, fixes belong upstream in BMAD-METHOD, scope discipline, evidence-before-claims.

**Tooling.** Use `rg` (ripgrep) and `fd` for filesystem search. **Never `find`.** Use `git` for drift detection rather than tree-walking. Use the episodic-memory MCP and the session journal corpus before any filesystem scan.

---

## Pre-Flight — Memory and Repo Introspection (mandatory; cheaper than scanning)

**Retrieval cost order — cheapest first.**
1. **Ask the user.** They hold the proximal surface of the problem and recall it directly. One short, targeted question is cheaper than any search.
2. **Episodic memory** (targeted queries derived from the user's pointers).
3. **Session journals** (only files named by episodic-memory hits).
4. **Repo introspection** (this BMAD-METHOD repo).
5. **Filesystem scans** in user projects (last resort).

---

**P-0. Ask the user before searching.** The user has tacit knowledge that no query will surface efficiently. Open with a small number of narrow questions — not a broad survey. Use AskUserQuestion (or an in-line terse question if AskUserQuestion is unavailable). Constraint: ≤ 4 questions, each ≤ 1 sentence, each with concrete options or a "skip" path.

Suggested questions (adapt — do not ask all four if the first two answer the rest):
- "Which of S1 (token usage), S2 (sloppiness), S3 (markdown) actually need investigation here, or are S1+S2 already fully closed by Evolutions 1/2/4?"
- "For S3, point me at the proximal sessions or commits in `~/repos/architecture/` where elements / FactWeave requirements derivation revealed the markdown limit. Date or filename is enough."
- "Are there specific session journal dates I should open directly, or skip the corpus entirely?"
- "Anything in the projects list (architecture, archbox, FactWeave, linkml, …) that is *not* worth chasing for this evolution wave?"

Capture the answers verbatim into `00-prior-context.md` under a `## User pointers` heading. These pointers **constrain** P-1 queries — do not run queries the user implicitly ruled out.

If the user declines to narrow, fall back to the broader P-1 plan below; otherwise treat the user's pointers as authoritative.

---

**P-1. Episodic memory — confirmatory, not exploratory.** Use the `episodic-memory:search-conversations` skill (or `mcp__plugin_episodic-memory__search` directly) **only with queries derived from P-0 user pointers, or for signatures the user did not narrow**. Use `episodic-memory:remembering-conversations` when a hit is ambiguous. Use `pro-workflow:replay-learnings` for prior corrections. Capture excerpts into `./fixes/discovery/00-prior-context.md` keyed by signature.

**Three primary signatures drive these queries.** Most BMAD pain reduces to one of three; everything else is downstream noise.

- **S1 — Workflow token usage** (largely already addressed by Evolution-1/2; verify and extend).
- **S2 — Workflow sloppiness** (largely already addressed by Evolution-1/2/4; verify and extend).
- **S3 — Markdown over-reliance** (the open foundational problem). Concrete locus: requirements for **elements** and **FactWeave** are derived **inside `/home/randyg/repos/architecture`** — that repo is where the markdown limitations bite hardest, where AsciiDoc + YAML alternatives originated, and where ADR rationale weakness and intermittent status YAML discipline first showed up.

```
queries (signature-grouped):

  # S1 — token usage / template bloat
  - "BMAD token inefficiency template"
  - "workflow preamble redundancy step file"
  - "BMAD context cost large prompt"

  # S2 — workflow sloppiness / discipline drift
  - "workflow state recovery stepsCompleted"
  - "workflow status writeback bmm-workflow-status"
  - "dev-story artifact mandatory if-exists"
  - "elicitation capture A/P/C facilitator"
  - "BMAD workaround _bmad-output ad-hoc"
  - "BMAD invariants duplication project"

  # S3 — markdown over-reliance, ADR weakness, YAML status discipline
  - "architecture repo elements requirements derivation"
  - "architecture repo FactWeave requirements"
  - "ADR rationale missing weak"
  - "ADR asciidoc yaml structure"
  - "asciidoc over markdown normative artifact"
  - "status update yaml discipline intermittent"
  - "sloppy note-taking BMAD project"
```

Each query produces zero or more hits. For each hit, capture: title, date, project, the one-sentence claim, and any specific file path or commit it cites. That index goes into `00-prior-context.md` — keyed by signature (S1/S2/S3) so Phase 5 dedup is trivial.

Directories with BMAD installed, ranked in order of bmad improvement/difficulties likelihood:

```
/home/randyg/repos/architecture/_bmad
/home/randyg/repos/kai/substrate/Projects/archbox/_bmad
/home/randyg/repos/kai/substrate/Plans/Factweave/_bmad
/home/randyg/repos/linkml/_bmad

/home/randyg/repos/kai/substrate/Plans/padimae/_bmad
/home/randyg/repos/kai/substrate/Projects/describe/_bmad
/home/randyg/repos/kai/substrate/Projects/claudefat/_bmad
/home/randyg/repos/teague/_bmad
/home/randyg/repos/relloq/_bmad
```

relevant projects with bmad references in ~/.claude/projects 
```
-home-randyg-repos-elmer
-home-randyg-repos-factweave-code
-home-randyg-repos-kai-substrate
-home-randyg-repos-kai-substrate-Packs-pydantic-perf
-home-randyg-repos-kai-substrate-Plans-ArchBox
-home-randyg-repos-kai-substrate-Plans-Factweave
-home-randyg-repos-kai-substrate-Plans-Factweave-V3
-home-randyg-repos-kai-substrate-Plans-padimae
-home-randyg-repos-kai-substrate-Projects-archbox
-home-randyg-repos-kai-substrate-Projects-archbox-docs
-home-randyg-repos-kai-substrate-Projects-claudefat
-home-randyg-repos-kai-substrate-Projects-describe
-home-randyg-repos-kai-substrate-Projects-rfp-triage
-home-randyg-repos-linkml
-home-randyg-repos-linkml-lkml
-home-randyg-repos-linkml-lkml--bmad
-home-randyg-repos-linkml-lkml-gen
-home-randyg-repos-relloq
-home-randyg-repos-teague
```

DUBIOUS relevance:
```
/home/randyg/repos/kai/substrate.wt/devprofile/Plans/padimae/_bmad
/home/randyg/repos/kai/issue-claudefat/Projects/archbox/docs/_bmad
/home/randyg/repos/kai/issue-claudefat/Plans/padimae/_bmad
/home/randyg/repos/linkml/.claude/worktrees/agent-a82db746/_bmad
/home/randyg/repos/linkml/.claude/worktrees/agent-a9634ca6/_bmad
/home/randyg/repos/linkml/.claude/worktrees/agent-a367f621/_bmad
/home/randyg/repos/linkml/.claude/worktrees/agent-ae35cf6f/_bmad
/home/randyg/repos/linkml/.claude/worktrees/agent-a1c3458d/_bmad
/home/randyg/repos/linkml/.claude/worktrees/agent-aab59c3f/_bmad
```


**P-2. Session journals — episodic-memory-driven pull, NOT a corpus grep.** `~/.claude/sessions/` holds 50+ dated `.tmp` files totalling >600 KB. Reading the corpus is wasteful; episodic memory **is** the index for it.

Procedure:
1. Take the P-1 hits. Each hit references a specific session date and project.
2. Build a small targeted file list: only the session files named by episodic memory hits.
3. Read those files specifically. **Do not** `rg` across the whole `~/.claude/sessions/` directory.
4. If a P-1 hit is vague about which session, ask episodic memory a refining query (`episodic-memory:remembering-conversations`) before opening any file.

Hard rule: every session file opened in P-2 must be justified by a citation from a P-1 hit. No fishing.

If P-1 produces zero hits for a signature, that signature has no journal evidence — log "no journal coverage for {Sx}" and move on. Do not compensate by scanning the corpus.

**P-3. This repo's substrate documentation.** Before discovering external uses, know what BMAD already provides.
- `./README.md` — top-level repo intro and pointers
- `./CLAUDE.md` and `./AGENTS.md` if present
- `src/modules/bmm/` directory tree (`fd -t d -d 3 . src/modules/bmm/`)
- All `SKILL.md` files in this repo (`fd -t f 'SKILL\.md' src/`).
- The bundle layout under whatever path BMAD installs from (look for `installer/`, `bundle/`, `dist/`, or similar in this repo's tree).
- BMAD's **canonical builder skills** (loaded via the `bmad-builder` plugin) — note them now; they are required tooling in Phase 6:
  - `bmad-builder:bmad-bmb-setup` — installs/configures a BMad module into a project (writes `_bmad/config.yaml`, `_bmad/config.user.yaml`, `_bmad/module-help.csv`).
  - `bmad-builder:bmad-module-builder` — plans, creates, validates BMad modules.
  - `bmad-builder:bmad-workflow-builder` — builds, converts, analyzes workflows and skills.
  - `bmad-builder:bmad-agent-builder` — builds, edits, analyzes Agent skills.
- Optional **problem-framing aids** for substrate decisions (Wave 1):
  - `bmad-creative-intelligence-suite:bmad-cis-problem-solving` (Dr. Quinn — TRIZ / Theory of Constraints / Systems Thinking) — useful when D1/D2/D3 decisions (markdown vs. AsciiDoc+YAML, ADR shape, requirements shape) are contested or have non-obvious tradeoffs.
  - `pro-workflow:plan-interrogate` — stress-test the integration plan once drafted (Phase 6).
  - `compound-engineering:document-review` — parallel persona review of `04-integration-plan.md` before execution.
- **Worktree dedup support**:
  - `compound-engineering:git-worktree` — manages worktrees and is the right reference for dedup logic in Phase 1.

**P-4. Confirm applied evolutions in this repo.** Do not infer from `./fixes/` README claims; verify via git.
```
git log --oneline --all -- src/ | rg -i 'evolution|state.recovery|writeback|dev-story|elicit'
git log --oneline -- ~/.claude/commands/bmad/dev-story.md   # evolution-4 target
git log --oneline a2c4d268 -1                                # evolution-4 commit
git log --oneline 8c4dcd87 -1                                # evolution-1+2 capture commit (note: only fixes/ tree, not src/)
git log --oneline fc2f5274 -1                                # related evolution-1 ancestor
```
Then for each evolution, **inspect the canonical workflow files** that should carry the patch and confirm or refute presence:
- Evolution-1: architecture and correct-course step files in `src/modules/bmm/workflows/...` — grep for the A/P/C menu, `stepsCompleted` frontmatter discipline, "FACILITATOR not content generator" language.
- Evolution-2: `step-01b-continue.md` files exist; `workflow.md` carries a State Recovery Check section.
- Evolution-3: final-step files write to `bmm-workflow-status.yaml`; template carries `create-epics-and-stories` row.
- Evolution-4: `dev-story` instructions say "Create or update story document (MANDATORY — always required)" rather than "if exists".

Record per evolution: `{applied-here | applied-elsewhere-only | spec-only | partial}` with the evidence path/line.

**P-5. Locate user's BMAD-using projects from memory + worktree info, not from `~/repos` recursion.**
- Episodic memory + sessions already names the projects (linkml, archbox, FactWeave, elements, …).
- Cross-reference `git config --global` for any aliases.
- Build a candidate list **first**, then verify presence by stat. Do not recurse `~/repos/`.

**Pre-Flight stop condition.** You can answer:
1. Which projects are BMAD-using? (named list, sourced from memory)
2. Which evolutions are actually applied to this repo's `src/`? (per-evolution verdict with evidence)
3. Which canonical `bmad-builder:*` skills will the integration plan invoke, and for which Phase 6 wave?
4. What journal entries (or `pro-workflow:replay-learnings` hits) describe known issues we have not yet catalogued?

Stop. Report. Wait for "proceed."

---

## Phase 1 — Worktree-Aware Project Inventory

Naive `fd _bmad ~/repos/` will read every worktree, every subagent copy, every `node_modules` cache. Do not do that. The `compound-engineering:git-worktree` skill is the reference for worktree handling — consult it before designing any traversal logic.

**1.1 Resolve canonical roots.** For each candidate project from Pre-Flight P-5:
```
cd <candidate>
git rev-parse --show-toplevel              # canonical root (skip if not a repo)
git worktree list                          # enumerate worktrees explicitly
```
Record the **primary working tree only** (the one whose path matches `--show-toplevel` from a fresh `cd`). All other worktrees are siblings — same `.git`, same history, ignore unless the user names one specifically.

**1.2 Skip set.** Build an exclusion list once:
```
EXCLUDES=(
  '.git/worktrees'      # secondary worktrees
  'node_modules'
  '.venv' 'venv' '__pycache__'
  'target' 'build' 'dist'
  '_bmad-output/.archive'   # if present
  '.claude/projects'    # subagent state copies
)
```
Use as `rg --glob '!{...}'` and `fd --exclude` arguments throughout.

**1.3 Confirm BMAD presence per primary root.** From inside the canonical root:
```
fd -t d -d 2 '^(_bmad|_bmad-output|bmad)$' . --exclude '.git/worktrees'
```
Project is BMAD-using if any hit. Also check `git ls-files | rg '^(_bmad|bmad)/'` to see whether BMAD state is **tracked** vs. ignored — this materially changes the drift-detection strategy in Phase 3.

**1.4 Inventory file.** Write `./fixes/discovery/01-project-inventory.md`:
```
| project | canonical_root | bmad_install_shape | bmad_tracked_in_git | last_bmad_activity | local_fixes_dir | branches_of_interest |
```
- `bmad_install_shape ∈ {full _bmad/, slim bmad/, hybrid, output-only}`
- `bmad_tracked_in_git ∈ {yes, no, partial}` — drives Phase 3 strategy
- `last_bmad_activity` = `git log -1 --format=%cs -- _bmad _bmad-output bmad 2>/dev/null` (one shell call)

**1.5 Explicitly elide:**
- This repo (`harness/skills/BMAD-METHOD.git`)
- Any worktrees of inventoried projects (covered by `git worktree list` skip)
- Any subagent-state directories under `.claude/projects/`

---

## Phase 2 — Issue Taxonomy

(unchanged from v1; reproduced for completeness)

**A. Workflow state & resumption** — A1 state recovery, A2 status writeback, A3 cross-session continuity, A4 worktree isolation violations.
**B. Artifact creation discipline** — B1 mandatory artifact skipping, B2 ad-hoc paths, B3 missing required sections, B4 numbering/naming drift.
**C. Elicitation & deliberation capture** — C1 unsolicited content, C2 A/P/C bypass, C3 `stepsCompleted` not updated, C4 deliberation loss.
**D. Document format & structure (substrate)** — D1 markdown over-reliance vs. AsciiDoc + YAML, D2 ADR shape inadequate, D3 requirements shape inadequate, D4 token bloat in templates, D5 prose where structured YAML belongs.
**E. Repository hygiene** — E1 BMAD invariant duplication, E2 slim/full install confusion, E3 config proliferation, E4 `.gitignore` gaps.
**F. Tooling gaps** — F1 local utility scripts, F2 manual ops, F3 missing/broken slash commands, F4 CLI ergonomics.
**G. Defects** — G1 step files crash/loop, G2 schema violations, G3 dangling cross-refs, G4 hard-coded paths.

Out-of-taxonomy → `05-open-questions.md`. Never expand silently.

---

## Phase 3 — Per-Project Discovery (git-first, scan-last)

For each project from Phase 1, in this order. Stop using tree diffs.

**3.1 Drift detection — fastest filter first.**

**Rule of thumb (pre-filter, applies to every project before git/hash work):** any customization of `_bmad/` method files will be **newer** than the creation date of `_bmad-output/` directory. 
Workflow runs read `_bmad/` and write to `_bmad-output/`. A pristine vendored install has `_bmad/` mtimes **at or before** `_bmad-output/`'s creation epoch (the install set both up together; nothing in `_bmad/` has been touched since). 
Files in `_bmad/` newer than `_bmad-output` ctime are customization candidates. This is limited to archbox, per user recollection confirmed by analysis.

N.B. - worktree copies of _bmad do not follow this pattern

```
cd $HOME/repos/kai/substrate/Projects
find archbox -type d -name _bmad | xargs -I % find % -type f -cnewer %-output 
```

- **Empty result** → `_bmad/` is likely pristine for this project; skip the rest of 3.1 unless the user named the project as a customization site.
- **Small result set** → those files are the customization shortlist. Run git/hash work (below) only on this shortlist, not the whole tree.
- **Cannot apply** (no `_bmad-output/`, or output recently bulk-regenerated): fall through to git/hash work below.

False-negative cases to flag in `05-open-questions.md` if encountered: bulk output regeneration after a period of `_bmad/` edits, gitignored output dirs that were freshly checked out, system clock skew during dispatch.

**3.1.a Drift via git, not via diff -r** (apply to the shortlist, or to the whole `_bmad/` if pre-filter was inapplicable):
- If `bmad_tracked_in_git == yes`:
  ```
  git log --oneline --since='12 months ago' -- _bmad/
  git diff origin/HEAD...HEAD -- _bmad/                # local divergence from remote
  git status -- _bmad/                                  # uncommitted edits
  ```
  Every commit and every uncommitted edit is a finding candidate. Read commit messages first; only read the diff for messages that look BMAD-relevant.
- If `bmad_tracked_in_git == no`:
  - The `_bmad/` directory is a vendored copy. Capture its provenance: look for an installer manifest, a `BMAD_VERSION` file, or a `.bmad-version` marker. If absent, hash a known-stable file (e.g. `_bmad/bmm/agents/dev.md`) and compare to this repo's canonical hash. Drift is discovered by *which files exist locally that aren't in canonical*, plus *which files diverge by hash*. One-shot script:
    ```
    (cd _bmad && fd -t f -H . | xargs sha256sum) > /tmp/proj-bmad.sha
    (cd ~/repos/harness/skills/BMAD-METHOD.git/<canonical-bundle-path> && fd -t f -H . | xargs sha256sum) > /tmp/canonical-bmad.sha
    diff <(sort /tmp/proj-bmad.sha) <(sort /tmp/canonical-bmad.sha)
    ```
  - Read only the diverged files.

**3.2 Local fix surface.**
```
fd -t d -d 3 '^(fixes|patches|evolutions|bmad-fixes|workarounds)$' . \
  --exclude '.git/worktrees' --exclude node_modules
```
Read every README/markdown under each. Classify per Phase 2.

**3.3 Ad-hoc workarounds in artifacts.**
```
rg -nE 'WORKAROUND|TODO\(BMAD\)|HACK|XXX|FIXME|HOTFIX|\bif exists\b|\bskip if\b|\bmanual step\b' \
  _bmad-output/ \
  --glob '!.git/worktrees/**' --glob '!**/node_modules/**'
```
Capture 5-line context per hit.

**3.4 Foundational format audits (one ripgrep per axis).**
- ADR audit:
  ```
  fd -t f -e md -e adoc -e asciidoc . --full-path | rg -i '/(adr|decision)' | head -20
  ```
  Sample 3. Score each against canonical shape (Status / Context / Decision / Consequences / Links). Note format and presence of structured YAML frontmatter.
- Requirements audit:
  ```
  fd -t f -e md -e adoc -e asciidoc . --full-path | rg -i '/(requirement|prd|spec)' | head -20
  ```
  Same.
- Token audit: identify the 3 largest BMAD step/template files the project actually invokes (cross-ref against `_bmad-output/` artifacts to know which were used). `wc -w`, estimate tokens (≈ words × 1.3). Flag duplicated preambles across step files (D4 evidence).

**3.5 Duplication audit (E1 — the big one).** This is where vendored BMAD copies that have *not* drifted are still findings: they shouldn't exist.
- Identify every file under `_bmad/` (or project-local `bmad/`) that is byte-identical to the canonical bundle in this repo. Those are pure duplication — candidates for symlink, submodule, or installer reference.
- Identify every file that drifted (already covered by 3.1) — those are the local-fix candidates.
- Output a 3-column table per project: `path | drift_status | recommendation` where recommendation ∈ {dedup-via-installer, upstream-the-fix, document-customization, delete-stale}.

**3.6 Tooling discovery.**
```
fd -e sh -e py -e js -e mjs -e ts -e Makefile . \
  --exclude '.git/worktrees' --exclude node_modules \
  | xargs rg -lE '\b(bmad|workflow|epic|story|adr)\b' 2>/dev/null
```
Per script: 1-line description, classify F1/F2/F3/F4.

**3.7 Journal pull.** Every project has its own session journals at `~/.claude/sessions/*-{project}-*.tmp`. Cross-reference:
```
rg -l --no-ignore -iE 'bmad|adr|workflow.*(broken|missing|hack)|workaround' \
  ~/.claude/sessions/ \
  | rg -i '<project-slug>'
```
Findings recorded in journals carry first-hand failure context — capture them with `git_refs: journal:<filename>`.

Per project, write `./fixes/discovery/02-{project-slug}.md` using the Phase 4 schema.

---

## Phase 4 — Evidence Schema (mandatory per finding)

```yaml
- id: F{NNN}
  project: {slug}                  # or "cross-project" for shared findings
  category: {A1|...|G4}
  title: {short imperative phrase}
  evidence:
    paths: [absolute or repo-relative]
    excerpt: |
      {3–15 lines of literal content}
    git_refs: [commits, branches, journal:<file>]
    memory_refs: [episodic-memory hit IDs if any]
  current_state: {how the project copes today}
  proposed_fix: |
    {one paragraph; references BMAD-METHOD canonical files;
     names the BMAD builder skill to use if creating new agent/workflow}
  dependency: {F-id | none}
  upstream_target: {repo-relative path of file to change in BMAD-METHOD}
  estimated_blast_radius: {file count + risk note}
  user_question: {explicit decision required, or null}
```

No evidence → discarded.

---

## Phase 5 — Synthesis & Deduplication

After per-project files exist:

1. Merge into `./fixes/discovery/03-master-catalogue.md`. **One row per unique finding**, N evidence rows when N projects exhibit it.
2. Cross-link evolution-{1..4}. Each is `complete | partial | superseded-by-{F-id} | expand-with-{F-id}`. Where Pre-Flight P-4 said "applied-elsewhere-only" or "spec-only," that evolution's F-row is *not done* — it is open work.
3. Foundational findings (D-category and E1) are tagged `tier: substrate`. They sequence first.
4. Dependency DAG rendered as Mermaid in the master catalogue.
5. Conflicts surface to `05-open-questions.md` as user decisions.

---

## Phase 6 — Integration Strategy onto `enhance/better-adr-capture`

Write `./fixes/discovery/04-integration-plan.md`.

**6.1 Sequencing principle.**
- Substrate before behavior: D-tier and E1 land before A/B/C.
- A1 (state recovery, Evolution-2 territory) before A2 (writeback, Evolution-3 territory).
- B/C land after substrate so they reference the new artifact shapes.
- F (tooling promotion) lands last — promote scripts only against stable upstream interfaces.

**6.2 Use BMAD's own builder skills.** Where the plan creates new workflows, agents, modules, or step files, prefer the canonical builders over hand-rolling:
- `bmad-builder:bmad-workflow-builder` — new or modified workflow scaffolding (use for upstreaming Evolution-1 step-file changes, Evolution-2 `step-01b-continue.md` patterns, Evolution-3 final-step writeback inserts).
- `bmad-builder:bmad-agent-builder` — new or modified Agent skills (use only if the integration plan requires a new agent — most evolutions modify existing workflows).
- `bmad-builder:bmad-module-builder` — when introducing a new BMad module (likely overkill for this evolution wave; flag in `05-open-questions.md` if proposed).
- `bmad-builder:bmad-bmb-setup` — only on user-projects after substrate ships, NOT on this BMAD-METHOD repo. Out of scope for the integration branch itself.
- `skill-creator:skill-creator` or `skill-create` — for any new non-BMad skill the plan introduces.

For substrate-tier framing (Wave 1, D-category):
- `bmad-creative-intelligence-suite:bmad-cis-problem-solving` — invoke Dr. Quinn for any substrate decision that the catalogue flags as contested or non-obvious (e.g., AsciiDoc-vs-Markdown precedence, ADR-shape canonicalization). Capture his output as decision rationale appended to the affected finding.

For plan validation before execution:
- `pro-workflow:plan-interrogate` — walk the decision tree of `04-integration-plan.md` one question at a time.
- `compound-engineering:document-review` — parallel persona review of `04-integration-plan.md` to surface role-specific gaps before any commit lands.

Hand-rolling new workflow structure when builder skills exist is a smell.

**6.3 Wave plan.** 3–5 waves. Each: single theme, atomic commits, defined smoke test, single-revert rollback.
Skeleton (derive content from catalogue):
- Wave 1 — Substrate: D2 ADR canonical (AsciiDoc + YAML option), D3 requirements shape, D1 format policy, D4 template token diet, D5 YAML promotion, E1 dedup pattern (installer/symlink decision).
- Wave 2 — State machinery: Evolution-2 upstream, Evolution-3 implementation incl. Fix D1/D2/D3, A3 journal handoff.
- Wave 3 — Artifact discipline: Evolution-1 upstream, Evolution-4 propagation across all "if exists" sites, B-category sweep.
- Wave 4 — Hygiene: E2, E3, E4.
- Wave 5 — Tooling promotion: F1 scripts → BMAD utilities, F3 missing slash commands, F4 ergonomics.

**6.4 Commit plan.** Per wave: commits in order. Conventional message: `<type>: <description>`. No mixing waves per commit.

**6.5 Validation gates.** Per wave:
- Run affected workflow end-to-end against a sandbox `_bmad-output/` (use a scratch project, not a real one)
- Verify `bmm-workflow-status.yaml` updates as designed
- Verify ADR/PRD output conforms to new shape
- Hash-diff sample user-project's `_bmad/` against new canonical — semantic regressions block, textual diffs OK

**6.6 Rollback plan.** Failed gate → `git revert <wave-range>`. No wave chains irreversible state.

**6.7 Out of scope (declare).**
- User-project working trees are **not** modified during integration
- No fix promoted without canonical BMAD restatement
- Slash-command surface in `~/.claude/commands/bmad/` is a separate workstream
- D1 enables AsciiDoc + YAML; it does not eliminate markdown

---

## Phase 7 — Deliverable Bundle

```
./fixes/discovery/
├── 00-prior-context.md         # episodic memory + journal extracts
├── 01-project-inventory.md
├── 02-{project-slug}.md        # one per BMAD-using project
├── 03-master-catalogue.md      # merged, deduplicated, DAG'd
├── 04-integration-plan.md      # waves, commits, gates, rollback
└── 05-open-questions.md        # user decisions required before execution
```

Surface `05-open-questions.md` to the user before any code is written on `enhance/better-adr-capture`. Per-question go-ahead is required.

---

## Hard Rules

1. **User first, search second.** P-0 (ask the user) is the cheapest retrieval. Treat user pointers as authoritative; episodic queries are confirmatory, not exploratory. Never burn a search budget the user could have answered in one sentence.
2. **Memory and journals before filesystem.** Pre-Flight P-1 / P-2 are non-optional, but P-2 only opens session files named by P-1 hits — never a corpus grep.
3. **`rg` and `fd` only.** No `find`. Always pass the `EXCLUDES` skip set.
3. **Canonical roots only.** `git rev-parse --show-toplevel` and `git worktree list` define the primary tree per project. Skip the rest.
4. **Elide BMAD-METHOD.** This repo is the substrate, never an analysis target.
5. **Mtime-pre-filter, then git, then hash.** Compare `_bmad/` file mtimes against `_bmad-output/` directory's **creation epoch** (birthtime via `stat %W`; ctime via `find -cnewer` as fallback) — NOT against the newest output artifact. Empty result = pristine, skip the rest. Then `git log` / `git diff` / `git status` on the shortlist. Hash-diff only when BMAD state is untracked or the pre-filter was inapplicable. **Caveat:** worktree copies of `_bmad/` do not follow this pattern; skip them.
6. **Read-only on user projects.**
7. **Verify, don't trust.** `./fixes/evolution-*/README.md` claims of "applied" must be reconfirmed via git log against `src/` paths in this repo.
8. **Use BMAD builder skills** when introducing new workflows/agents.
9. **Substrate before behavior.** D-tier and E1 ship first.
10. **Phase gates are real.** Stop after Pre-Flight, Phase 1, Phase 5. Wait for "proceed."
11. **No fake workflow execution.** Patch canonical files in BMAD-METHOD; do not pretend a workflow ran.
12. **Restate user priorities before tradeoffs.**

---

## Kickoff

Begin Pre-Flight. Report when P-1 through P-5 are complete with the four-question stop-condition answered. Do not advance until told to.
