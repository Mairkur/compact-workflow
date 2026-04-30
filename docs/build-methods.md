# Build Methods — compact-workflow

Two ways to maintain fork. Score /100. Pick fit current profile.

## Profile

- Solo dev
- ~5 improvements planned
- Manual symlink install (`~/.claude/skills/compact-workflow → workflow/`)
- Upstream sync: occasional, manual
- No CI build step

## Method A — Inline edits + git merge (current)

Edit `workflow/` files directly. Pull upstream via `git merge upstream/main --allow-unrelated-histories` (first time) or `git merge upstream/main` (after). Resolve conflicts inline.

```
compact-workflow/
  workflow/          ← edited in place; symlink target
  CHANGES.md         ← divergence ledger
  NOTICE.md          ← attribution + upstream SHA
  specs/             ← spec history
```

### Scoring

| Criterion | Score | Why |
|-----------|-------|-----|
| Simplicity | 9/10 | One tree, edit, commit, done |
| Reliability | 9/10 | Symlink read direct by Claude — no build step to fail |
| Friction | 10/10 | Zero ceremony per change |
| Tooling fit | 10/10 | Git native — merge, blame, log all work |
| Maintainability | 5/10 | Edits scattered across upstream files; hard to extract later |
| Conflict handling | 5/10 | Append-only sections help, but inline edits collide |
| Auditability | 6/10 | CHANGES.md + commits — but improvement boundaries fuzzy |

**Total: 77/100**

### When break

- Improvements > 10 (edits sprawl)
- Sync conflicts > 15 min/cycle (manual cost compounds)
- Multi-dev contributors (need clear boundaries)

## Method B — Build pipeline (upstream/ + improvements/ → dist/)

Keep upstream pristine in `upstream/`. Store improvements as overlay patches/files in `improvements/`. Build script merges → `dist/workflow/`. Symlink target `dist/workflow/`.

```
compact-workflow/
  upstream/workflow/      ← pristine clone, never edit
  improvements/           ← overlays (patches, append-files, replacement files)
  dist/workflow/          ← built output; symlink target
  build.sh                ← upstream + improvements → dist
  CHANGES.md
```

### Scoring

| Criterion | Score | Why |
|-----------|-------|-----|
| Maintainability | 9/10 | Improvements isolated; trivial to list/remove |
| Conflict handling | 10/10 | Upstream sync = `git pull` in `upstream/`; rebuild — no manual merge |
| Auditability | 9/10 | Each improvement = one file/patch; diff vs upstream = `dist - upstream` |
| Friction | 5/10 | Edit improvement → run build → test (extra step) |
| Reliability | 6/10 | Build script = new failure surface; stale `dist/` risk |
| Simplicity | 5/10 | Three trees + script to reason about |
| Tooling fit | 4/10 | Patch tooling fragile; non-git mental model |

**Total: 69/100**

### When wins

- Improvements > 10
- Upstream changes hit improvement areas often
- Want to publish improvements as standalone overlays
- Multi-dev (clear file ownership)

## Verdict

**Method A wins for current profile (77 vs 69).**

Reason: 5 improvements, solo dev, no CI. Build pipeline overhead (Method B) costs more than it saves. Method A's git-native flow + symlink-direct-read = lowest friction path.

### Crossover point

Switch to Method B when:
- 10+ improvements active, OR
- Upstream sync conflicts > 15 min/cycle, OR
- Multi-dev contributors, OR
- Want to publish improvements as overlay package

## Hybrid (recommended)

Stay Method A. Defer crossover by:

1. **Prefer new files over edits** — improvement = new file in `workflow/references/`. Append-only references > inline edits. No conflict possible.
2. **Append-only sections in upstream files** — `## Tone` at bottom of `SKILL.md` (caveman spec pattern). Upstream edits header; we own footer.
3. **Track improvement count** — when 8th improvement lands, plan migration to Method B.
4. **Keep CHANGES.md tight** — one line per improvement, link to spec. Audit trail without ceremony.

This way Method A scales further before crossover triggers.

## How Claude reads it (Method A)

1. Symlink `~/.claude/skills/compact-workflow → ~/Documents/GitHub/compact-workflow/workflow/`
2. Claude scans `~/.claude/skills/*/SKILL.md` on session start
3. Reads `workflow/SKILL.md` direct — no build, no copy
4. Edit `workflow/SKILL.md` → next session sees change immediately

No `dist/`. No build. Edit = live.
