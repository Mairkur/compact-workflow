# Divergences from upstream agent-skills/workflow

Future merger reads this to resolve conflicts with correct intent. Also feeds Method B migration (see `docs/build-methods.md`) — `extractable: yes` entries map straight to overlay files.

## Entry format (improvements after bootstrap)

```
- <improvement-name> — spec: specs/active/<file>.md
  - <file>: <type> | anchor: <marker> | extractable: yes|no | <reason>
```

- **type**: `NEW` | `EDIT-frontmatter` | `EDIT-body` | `APPEND` | `COPY`
- **anchor**: section heading or line range for splice edits; `n/a` for `NEW`/`APPEND`/`COPY`
- **extractable**:
  - `yes` — append-only, new file, or copy. Drop straight into Method B `improvements/`.
  - `no` — inline splice. Needs anchor + manual overlay on migration.
- **reason**: one-line why

Bootstrap section below predates this format — kept as-is. Apply new format from caveman tone forward.

## Bootstrap (commit 2) — forked from 5f6f937ad7e0bd02d8cbfbf4e3db6e460f10e469

- workflow/SKILL.md (EDIT-frontmatter): name → compact-workflow, description + keyword list (compact <cmd> phrases only), version → 0.1.0, upstream metadata added
- workflow/SKILL.md (EDIT-body): title → "Compact Workflow", Action Router input patterns → compact <cmd> form, Status Action output examples → compact <cmd> form
- workflow/README.md (EDIT-body): title, install (symlink), attribution, all command examples → compact <cmd>, /compact builtin caveat added
- workflow/references/reviews/core-portable-review-spec.md (EDIT-body, lines 3+174): "workflow" skill self-refs → "compact-workflow"
- workflow/references/reviews/executor-patterns.md (EDIT-body, line 63): "core workflow skill" → "core compact-workflow skill"
- workflow/references/templates/status-output.md (EDIT-body, line 3): "`workflow` / status check" → "`compact-workflow` / status check"
- NOTICE.md (NEW, repo root): fork attribution and upstream SHA
- CHANGES.md (NEW, repo root): this file
- LICENSE (COPY, repo root): copied verbatim from agent-skills root

### Preserved as-is (case-by-case decision)
- workflow/references/codebase-intelligence.md line 107: "Never block workflow on CI" — generic concept, not skill name
- workflow/references/session-management.md line 105: "you know the workflow" — generic concept
- workflow/references/actions/review.md line 39: "workflow-level review policy" — generic concept

## Future improvements

- caveman-tone — spec: specs/shipped/2026-04-30-caveman-tone.md | shipped: 2026-04-30 | source: JuliusBrussee/caveman@84cc3c14fa1e10182adaced856e003406ccd250d
  - workflow/SKILL.md: APPEND | anchor: `## Tone` (after References section) | extractable: yes | inline caveman rules + override clause + auto-clarity trigger list + spec-file boundary + persistence clause
  - workflow/references/tone/caveman.md: NEW | anchor: n/a | extractable: yes | source attribution + caveman SHA + drift policy + local divergences from upstream caveman
  - workflow/README.md: APPEND | anchor: before `## Attribution` section | extractable: yes | caveat: caveman default + how to disable + boundaries
  - CHANGES.md: EDIT-body | anchor: this entry | extractable: n/a | self-reference (housekeeping)

- skill-quality-guard — spec: specs/shipped/2026-04-30-skill-quality-guard.md | shipped: 2026-04-30 | source: internal (synthesized from caveman + bootstrap retros + portable review spec) | quality-guard-version: v1
  - workflow/references/quality-guard.md: NEW | anchor: n/a | extractable: yes | full rules table (QG-1 through QG-20) + tier-conditional enforcement + detection rule + pre-fill heuristic + verdict rubric + violation report format + version v1
  - workflow/references/spec-template.md: EDIT-body | anchor: `## Validation Rules` table (rule 15 added) + new conditional `## Quality Guard Results` section template + frontmatter `quality-guard-version` field | extractable: no | adds Quality Guard validation rule + conditional section template
  - workflow/references/actions/plan.md: EDIT-body | anchor: header (load directive) + new `## Step 3.7: Quality Guard Detection` between Step 3.5 and Step 4 + Step 6 Readiness Algorithm | extractable: no | detection + render + tier-conditional + dev-readiness block
  - workflow/references/actions/spec-review.md: EDIT-body | anchor: header (load directive) + Step 2 (auditor as REQUIRED 4th perspective) + Step 6 (merge updates Quality Guard Results) + new `### Step 7: Quality Guard Auditor` | extractable: no | auditor perspective + BLOCKING verdict integration
  - workflow/references/actions/ship.md: EDIT-body | anchor: header (load directive) + new `## Phase 0a: Quality Guard Pre-Iteration Gate` before Phase 0 | extractable: no | pre-iteration BLOCKING gate, no `--force`
  - workflow/SKILL.md: APPEND | anchor: `## Quality Guard` after `## Tone` section | extractable: yes | governance link + tier summary + version pin reference
  - workflow/README.md: APPEND | anchor: before `## Tone` section | extractable: yes | user-facing caveat: gates skill-improvement specs, what it checks, no disable, version pinning
  - CHANGES.md: EDIT-body | anchor: this entry + new `### Grandfathered (pre-Quality-Guard)` subsection | extractable: n/a | self-reference + retroactive grandfather log

### Grandfathered (pre-Quality-Guard)

These improvements shipped before the Quality Guard existed (introduced 2026-04-30 by `skill-quality-guard` spec). Logged here as audit-only — no retroactive enforcement.

- bootstrap (commit `5085606` + `db787b3`) — fork from agent-skills/workflow@5f6f937. Pre-Guard. Audit notes: source attribution recorded (QG-1 satisfied retroactively). Router preserved on first ship (QG-4 satisfied). No tone overlay introduced (QG-2 N/A). No external rule sources beyond agent-skills (QG-1 satisfied).
- caveman-tone (commit `7f8f824`) — caveman tone overlay v1. Pre-Guard. Audit notes: source attribution recorded with caveman@84cc3c1 (QG-1 satisfied retroactively). Document boundary documented in SKILL.md `## Tone` and `references/tone/caveman.md` (QG-2 satisfied). Auto-clarity trigger list explicit (QG-3 satisfied). No router edits (QG-4 N/A). Inline rules — no broken file references (QG-7 satisfied — caveman tone v1 spec-review caught loading mechanism issue; current shipped form is inline-only).
