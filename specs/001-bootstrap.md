# Spec: compact-workflow bootstrap

**Status:** Todo
**Owner:** mairkur
**Created:** 2026-04-30
**Revised:** 2026-04-30 (post spec-review #2)

## Goal

Fork `agent-skills/workflow` skill into standalone repo `compact-workflow`. Rename skill, add `compact <subcommand>` phrase-based keyword detection. Coexist with upstream `workflow` skill. No behavior changes yet — improvements deferred.

## Context

- Source: `~/Documents/GitHub/agent-skills/workflow/` (local clone of agent-skills repo, MIT)
- Target: `~/Documents/GitHub/compact-workflow/` (local only, no GitHub remote yet)
- Layout: **preserve `workflow/` subdir** so paths match upstream (enables native `git merge upstream/main`). Repo root holds meta files (LICENSE, NOTICE, CHANGES, specs); skill files live under `workflow/`.
- Install: symlink the **inner** `workflow/` dir as the skill — `ln -s ~/Documents/GitHub/compact-workflow/workflow ~/.claude/skills/compact-workflow`
- Trigger style: `compact <subcommand>` phrases — e.g. `compact spec`, `compact ship`, `compact fix`, `compact done`. Skill fires on phrase match in user prompt.

## Trigger model (decided)

Skill activation = keyword match against user prompt via SKILL.md `description` field. No slash command file. User invokes by typing prompts like:

- `compact spec {idea}` → route to plan action
- `compact ship` → ship action
- `compact fix {bug}` → fix action
- `compact review` → review action
- `compact spec-review` → spec-review action
- `compact focus` / `compact done` / `compact drop` / `compact spike`
- `compact workflow` / `compact status` → status action

Keyword list = phrases prefixed with `compact `, plus bare `compact-workflow`. **No bare `compact`** (collide with `/compact` builtin + too broad). **No bare subcommand words** (collide with upstream `workflow` skill).

## Coexistence with upstream `workflow`

Both skills installable side-by-side. Disambiguation via prefix:
- `plan X` → upstream workflow
- `compact spec X` → compact-workflow

Fork keyword list MUST drop all bare subcommand triggers (`plan`, `ship`, `fix`, `done`, `drop`, `spike`, `review`, `focus`, `workflow`, `what's next`, etc.). Only `compact <…>` phrases trigger.

## Out of scope (future specs)

- Caveman-style shorter answers
- Linear issue auto-update per workflow step (spec, spec-review, ship, …)
- Plugin marketplace entry
- Improvements to spec template, quality gates, etc.
- Slash command file `~/.claude/commands/compact-workflow.md` (revisit if keyword match unreliable)

## Upstream reconciliation strategy

Fork must stay mergeable with upstream `agent-skills/workflow` long-term. Pattern:

**Tracked fork with upstream remote — preserve `workflow/` subdir for path-matching merges.**

```
compact-workflow/  (git repo, init at bootstrap)
  ├─ workflow/                  ← skill files, paths mirror upstream
  │   ├─ SKILL.md
  │   ├─ README.md
  │   └─ references/
  ├─ LICENSE
  ├─ NOTICE.md
  ├─ CHANGES.md
  ├─ specs/
  └─ remote upstream → ~/Documents/GitHub/agent-skills (or origin URL)
  └─ remote origin   → user-controlled remote (added later)
```

Path mirroring (fork `workflow/SKILL.md` ↔ upstream `workflow/SKILL.md`) lets `git merge upstream/main` apply changes directly — no subtree, no manual port. Conflicts surface only on lines edited in commit 2+. Skill loader sees the symlink target (inner `workflow/`), unaware of repo wrapping.

### Commit shape (atomic divergence)

```
commit 1: pristine import      ← verbatim copy of upstream workflow/, zero edits
commit 2: bootstrap rename     ← all rename + keyword + body-rewrite edits, single commit
commit 3..N: improvements      ← each improvement isolated to its own commit
```

Rationale: pristine first commit = clean diff base. Future `git merge upstream/main` only conflicts on lines touched in commit 2+. Atomic rename commit makes conflict surface predictable.

### Conflict-minimization tactics (apply to all future improvement specs)

1. **New files over edits.** Improvements that can live in new files (e.g. `references/integrations/linear.md`, `references/tone/caveman.md`) cause zero merge conflict.
2. **Append-only edits.** When modifying upstream file, append new section at bottom — upstream rarely appends bottom-of-file.
3. **Localize SKILL.md edits.** Frontmatter (name, keywords, version) WILL conflict every sync — accept it. Group body edits into clearly bounded sections.
4. **`CHANGES.md` ledger.** Every divergence documented + reason. Future merger reads it to resolve conflicts with correct intent.
5. **Tag each sync.** `git tag upstream-sync-<date>` after merge. Revert path if merge breaks skill.

### Sync workflow (run periodically)

```bash
git fetch upstream
git log upstream/main --oneline ^HEAD          # review upstream changes
git merge upstream/main                         # resolve conflicts using CHANGES.md
# fresh Claude Code session, run smoke tests (validation 6-7)
git tag upstream-sync-$(date +%Y-%m-%d)
```

Symlink does not need re-creating each sync — points at `workflow/` dir, content updates in place.

If upstream is local-only repo (not git), capture upstream tarball + date instead of fetch — degrade gracefully.

## Deliverables

1. Repo dir `compact-workflow/` with `workflow/` subdir (skill files mirroring upstream paths)
2. `workflow/SKILL.md` frontmatter: `name: compact-workflow`
3. Description with phrase-based auto-activation keywords (see Trigger model)
4. `workflow/README.md` rewritten: title, install path, invocation examples use `compact <cmd>` form
5. `LICENSE` copied from agent-skills repo root to compact-workflow root (MIT, attribute upstream)
6. `NOTICE.md` (repo root) crediting upstream source + commit SHA (or date+URL fallback)
7. Body text rewrite: replace literal `workflow` skill self-references in `workflow/SKILL.md` + `workflow/references/` with `compact-workflow` for consistency. Preserve `workflow` only in historical/cross-skill prose where rename would break meaning.
8. **Action Router rewrite (SKILL.md body):** input patterns updated from bare subcommand words (`"plan"`, `"ship"`, …) to `"compact <cmd>"` phrases — must match the activation keywords. Internal action file paths unchanged (still `references/actions/plan.md`, etc.).
9. Caveat in README: `/compact` is a Claude Code builtin (transcript compression). Do not type `/compact` to invoke this skill — use `compact <subcommand>`.
10. `git init -b main` at repo root + upstream remote + atomic commit shape (pristine import → bootstrap rename) per Upstream reconciliation strategy.
11. `CHANGES.md` (repo root) ledger documenting every divergence from upstream + reason.

## File-by-file plan

### Copy structure
- `agent-skills/workflow/` → `compact-workflow/workflow/` (full subdir, structure preserved — SKILL.md, README.md, references/**)
- `agent-skills/LICENSE` → `compact-workflow/LICENSE`

### Modify workflow/SKILL.md frontmatter

```yaml
name: compact-workflow
description: |
  Compact variant of the workflow skill (forked from agent-skills/workflow).
  Same 10 actions, invoked via "compact <subcommand>" prefix to coexist with upstream workflow.
  Auto-activates on:
  "compact spec", "compact plan", "compact spike",
  "compact ship", "compact implement", "compact build",
  "compact fix", "compact debug", "compact repair",
  "compact review", "compact spec-review", "compact challenge spec",
  "compact focus", "compact prioritize",
  "compact done", "compact finish", "compact drop", "compact abandon",
  "compact workflow", "compact status", "compact what's next",
  "compact-workflow".
license: MIT
compatibility: "Agent-agnostic. Works with any agent that can read SKILL.md files and project rule conventions."
metadata:
  version: "0.1.0"
  upstream: "agent-skills/workflow"
  upstream_version: "1.2"
  forked_from: "<sha-or-date>"   # SUBSTITUTE with value captured in Step 1 — no literal angle brackets in committed file
```

Note: dropped `"compact next"` (false-trigger risk on phrases like "compact next month"); kept `"compact what's next"` instead.

### Modify workflow/SKILL.md body

- Title `# Workflow` → `# Compact Workflow`
- Self-references "workflow skill" → "compact-workflow skill" (consistency)
- **Action Router rewrite** (lines ~92-110 in upstream): replace input patterns with `compact <cmd>` phrases. Action file targets unchanged. Example:
  ```
  ├─ "compact spec", "compact plan"          → references/actions/plan.md
  ├─ "compact spike", "compact explore"      → references/actions/spike.md
  ├─ "compact ship", "compact implement"     → references/actions/ship.md
  ├─ "compact fix", "compact debug"          → references/actions/fix.md
  ├─ "compact review"                        → references/actions/review.md
  ├─ "compact spec-review"                   → references/actions/spec-review.md
  ├─ "compact focus", "compact prioritize"   → references/actions/focus.md
  ├─ "compact done", "compact finish"        → references/actions/done.md
  ├─ "compact drop", "compact abandon"       → references/actions/drop.md
  └─ "compact workflow", "compact status",
     "compact what's next"                   → Status Action (below)
  ```
- Status Action output examples: rewrite (`Run: plan {idea}` → `Run: compact spec {idea}`, `Run: ship` → `Run: compact ship`, etc.)
- Commands table (lines ~38-50): leave action names (`plan`, `ship`, …) as-is — these are conceptual labels, not user inputs.

### Modify workflow/references/ body text

Files with literal `workflow` self-refs (per spec-review item 4):
- `workflow/references/reviews/core-portable-review-spec.md` (lines 3, 174)
- `workflow/references/reviews/executor-patterns.md` (lines 60, 63)
- `workflow/references/actions/review.md` (line 39)
- `workflow/references/templates/status-output.md` (line 3)
- `workflow/references/codebase-intelligence.md` (line 107)
- `workflow/references/session-management.md` (line 105)

Rewrite rule: when "workflow" refers to **this skill itself**, rewrite → `compact-workflow`. When it refers to generic concept ("workflow on CI"), leave alone. Each file inspected case-by-case.

### Modify workflow/README.md
- Title → `# compact-workflow`
- Install section: `ln -s ~/Documents/GitHub/compact-workflow/workflow ~/.claude/skills/compact-workflow`
- Attribution paragraph + link to upstream agent-skills repo
- Invocation examples → `compact spec {idea}`, `compact ship`, `compact fix {bug}`
- Caveat block: `/compact` is Claude Code builtin — do not confuse

### New files
- `NOTICE.md` — content template:
  ```
  # NOTICE

  compact-workflow is a fork of the `workflow` skill from agent-skills.

  - Upstream: agent-skills repository (local: ~/Documents/GitHub/agent-skills)
  - Upstream version: 1.2
  - Forked at: <SHA or date 2026-04-30>
  - License: MIT (preserved, see LICENSE)

  Modifications from upstream:
  - Renamed skill to `compact-workflow`
  - Activation switched to `compact <subcommand>` phrase prefix to coexist with upstream
  - Updated user-facing examples and self-references
  ```
- `CHANGES.md` — divergence ledger. Initial content:
  ```
  # Divergences from upstream agent-skills/workflow

  Each entry: file, type (NEW | EDIT-frontmatter | EDIT-body | APPEND), reason.
  Future merger reads this to resolve conflicts with correct intent.

  ## Bootstrap (commit 2)
  - workflow/SKILL.md (EDIT-frontmatter): name, description+keywords, version metadata
  - workflow/SKILL.md (EDIT-body): title, Action Router patterns (compact <cmd>), status output examples, self-references
  - workflow/README.md (EDIT-body): title, install, attribution, examples, /compact caveat
  - workflow/references/reviews/core-portable-review-spec.md (EDIT-body): self-refs to compact-workflow
  - workflow/references/reviews/executor-patterns.md (EDIT-body): self-refs
  - workflow/references/actions/review.md (EDIT-body): self-refs
  - workflow/references/templates/status-output.md (EDIT-body): self-refs
  - workflow/references/codebase-intelligence.md (EDIT-body): self-refs (case-by-case)
  - workflow/references/session-management.md (EDIT-body): self-refs (case-by-case)
  - NOTICE.md (NEW, repo root): attribution
  - CHANGES.md (NEW, repo root): this file
  - LICENSE (COPY, repo root): from agent-skills root

  ## Future improvements
  - <append entries here per improvement spec>
  ```
- `specs/001-bootstrap.md` (this file)

## Validation

1. `grep -rn "^name: workflow$" compact-workflow/workflow/` → 0 results (anchored frontmatter check)
2. `grep -rn "^name: compact-workflow$" compact-workflow/workflow/SKILL.md` → 1 result
3. `grep -rn "compact spec" compact-workflow/workflow/SKILL.md` → keyword present
4. `grep -rn '"compact"' compact-workflow/workflow/SKILL.md` → 0 results (no bare `compact` keyword leaked)
5. `grep -rn "compact-workflow" compact-workflow/workflow/SKILL.md` → multiple results (frontmatter + body)
6. Diff `compact-workflow/workflow/references/` vs `agent-skills/workflow/references/` → only intentional self-reference rewrites differ; structure identical
7. **Live trigger (upstream installed):** symlink `~/Documents/GitHub/compact-workflow/workflow` → `~/.claude/skills/compact-workflow`. Fresh Claude Code session, type `compact spec test idea` → compact-workflow activates and routes to plan action. Type `plan test idea` → only upstream `workflow` activates (no compact-workflow double-fire). **Prerequisite: upstream `workflow` skill must be installed in parallel.**
8. **Live trigger (upstream absent):** if user uninstalled upstream, `compact spec X` still activates compact-workflow; bare `plan X` activates nothing (or whatever else matches).
9. **False-trigger check:** type prompt with bare `compact` (e.g. "compact this JSON file") → compact-workflow does NOT activate.
10. **Merge feasibility:** after commit 2, run `git merge upstream/main --no-commit --no-ff` against current upstream HEAD. Expected: clean merge (or conflicts only on commit-2 lines). Abort with `git merge --abort` after confirming. Validates reconciliation strategy actually works.

## Steps

1. **Capture upstream reference.** Try `git -C ~/Documents/GitHub/agent-skills rev-parse HEAD`. Save output as `UPSTREAM_REF`. If command fails (not a git repo), set `UPSTREAM_REF="date-2026-04-30"` and note in NOTICE.md that sync is manual-tarball-based (no `upstream` remote possible). This single value substitutes:
   - commit 1 message: `chore: import workflow skill from agent-skills@${UPSTREAM_REF}`
   - NOTICE.md `Forked at:` field
   - SKILL.md frontmatter `forked_from:` field
2. **Copy structure.** Make repo root `compact-workflow/`. Copy `agent-skills/workflow/` → `compact-workflow/workflow/` (preserve subdir). Copy `agent-skills/LICENSE` → `compact-workflow/LICENSE`. **Do not** copy or move the existing `compact-workflow/specs/` (already created); leave it alone — it is excluded from commit 1.
3. **Initialize git on main branch.** `git init -b main` at `compact-workflow/`. **Commit 1 (pristine import) — explicit stage list:**
   ```
   git add workflow/ LICENSE
   git commit -m "chore: import workflow skill from agent-skills@${UPSTREAM_REF}"
   ```
   `specs/` and any other meta files NOT staged. Commit 1 must contain only mirror of upstream paths — clean diff base.
4. **Add upstream remote.** If agent-skills is git: `git remote add upstream ~/Documents/GitHub/agent-skills` (or its origin URL). Verify with `git fetch upstream`. If not git, skip and record fallback in NOTICE.md.
5. Edit `workflow/SKILL.md` frontmatter (name, description+keywords, version metadata). **Substitute `<sha-or-date>` placeholder with `${UPSTREAM_REF}` — no literal angle brackets in the saved file.**
6. Edit `workflow/SKILL.md` body (title, Action Router input patterns → `compact <cmd>` form, self-refs, status output examples).
7. Walk reference files listed above; rewrite self-refs case-by-case.
8. Rewrite `workflow/README.md` (title, install cmd pointing at inner `workflow/`, attribution, examples, `/compact` caveat).
9. Write `NOTICE.md` (repo root) from template, with `${UPSTREAM_REF}` substituted.
10. Write `CHANGES.md` (repo root) from template (bootstrap section pre-populated, paths prefixed `workflow/`).
11. Run validation 1–6 (grep + diff).
12. **Commit 2 (atomic bootstrap rename) — explicit stage list:**
    ```
    git add workflow/ NOTICE.md CHANGES.md specs/
    git commit -m "feat: rename to compact-workflow + phrase-prefix keywords"
    ```
    Single commit. Conflict surface for future upstream merges = exactly this commit's diff.
13. Symlink install + run validation 7–9 (live trigger smoke tests).
14. Run validation 10 (merge feasibility dry-run): `git fetch upstream && git merge upstream/main --no-commit --no-ff` then `git merge --abort`. Confirms strategy works.
15. (DEFERRED — ask user) Add user `origin` remote + push, when ready.

## Open questions

- None blocking after spec-review #2.
- Re-evaluate slash command file if validation 7 fails (keyword match unreliable for phrase form).
- If validation 10 (merge dry-run) reveals upstream-side path drift in future, revisit option (a) `git subtree`.
- Pre-existing dir at `~/.claude/skills/compact-workflow/` (not symlink) will trip install — verify target is empty / non-existent before `ln -s`.

## References

- Source skill: `~/Documents/GitHub/agent-skills/workflow/SKILL.md`
- Source LICENSE: `~/Documents/GitHub/agent-skills/LICENSE`
- Skill loader: `~/.claude/skills/<name>/SKILL.md`
