# Quality Guard

> **Version: v1.1**
> **Status: AUTHORITATIVE — loaded by `plan`, `spec-review`, `ship` actions when spec is classified as skill-improvement.**
> **Spec source: `specs/shipped/2026-04-30-skill-quality-guard.md`**
> **v1.1 corrigendum:** post-ship smoke audit added CHANGES.md-only pre-fill row + clarified QG-7 evidence narration. See `## Changelog` at end of file.

The Quality Guard is a mandatory rule set that gates every spec which modifies the compact-workflow skill itself. Its purpose: prevent silent quality regressions in the fork (router corruption, narrowed security boundaries, broken loading mechanisms, traceability loss, source-attribution drift, etc.).

Improvements to user code (feature specs) are NOT subject to the Quality Guard.

---

## Detection Rule

A spec is a **skill-improvement spec** if its `Codebase Impact` table contains ANY of:

- Path under `workflow/` (e.g. `workflow/SKILL.md`, `workflow/references/**`, `workflow/README.md`)
- Repo-root file: `CHANGES.md`, `NOTICE.md`, `LICENSE`
- Path under `docs/` (e.g. `docs/build-methods.md`)

Path under `specs/` is NOT a trigger — specs about features (or specs about specs) are still feature work, not skill changes.

**Hybrid spec** (touches both skill paths AND user-code paths): classified as skill-improvement (stricter posture wins).

If detection matches, the agent MUST:
1. Render the `## Quality Guard Results` section into the spec (using template below)
2. Pre-fill verdicts for trivially-N/A rules (heuristic below)
3. Apply tier-conditional enforcement (table below)
4. Block dev-readiness until the section is GUARD-CLEAN

---

## Tier-Conditional Enforcement

| Spec tier | Rules enforced (BLOCKING) | Rules informational |
|-----------|--------------------------|---------------------|
| `mini` (skill-improvement) | Tier 1 only (QG-1 → QG-10, 10 rules) | Tier 2 (audited but not blocking), Tier 3 (always informational) |
| `standard` (skill-improvement) | Tier 1 + Tier 2 (QG-1 → QG-18, 18 rules) | Tier 3 |
| `trivial` / `micro` (skill-improvement) | Tier 1 only — same as mini | Tier 2 + Tier 3 |
| Feature spec (any tier) | None (Quality Guard not applied) | All |

Tier 3 rules (QG-19, QG-20) are always informational. They never block.

---

## Pre-Fill Heuristic

The agent auto-fills `NOT_APPLICABLE` for rules that are trivially N/A based on file types touched. User reviews + confirms (cannot silently bypass).

| Rule | Auto-N/A condition |
|------|-------------------|
| QG-1 | No external source (caveman, agent-skills, etc.) referenced in spec → N/A |
| QG-4 | Spec does NOT touch `workflow/SKILL.md` Action Router section AND no router files edited → N/A |
| QG-7 | Spec does NOT add file references in SKILL.md → N/A |
| QG-12 | Spec does NOT introduce stateful overlay (tone mode, level, mode switch) → N/A |
| QG-17 | Spec does NOT add new trigger phrases or keywords → N/A |
| QG-19 | Always evaluated (count check) — never auto-N/A |
| QG-20 | Always evaluated (this rule = the meta-check) — never auto-N/A |

**CHANGES.md-only specs** (housekeeping log entries; only `CHANGES.md` modified, no `workflow/` paths): auto-N/A QG-2 through QG-8 and QG-11 through QG-17 by definition. Active rules reduce to QG-1, QG-9, QG-10, QG-18, QG-19, QG-20 (6 active vs 18 default for standard tier). User confirms during spec-review (cannot pass silently). This relief applies to the spec where `Codebase Impact` lists `CHANGES.md` as the sole skill-path file.

User MUST confirm pre-filled N/A values during spec-review (cannot pass silently).

---

## Verdict Vocabulary

Each rule receives one verdict:

- **`PASS`** — rule satisfied; evidence cited (file path or section anchor)
- **`WARN`** — rule deviation justified in spec; recorded but not blocking (Tier 2 only; Tier 1 cannot WARN)
- **`FAIL`** — rule violated; no justification accepted. BLOCKING for Tier 1 + Tier 2.
- **`NOT_APPLICABLE`** — rule does not apply; reason cited (e.g. "No external source — QG-1 N/A")

**A skill-improvement spec is GUARD-CLEAN when:**
- Every Tier 1 rule is `PASS` or `NOT_APPLICABLE` (no FAIL, no WARN — Tier 1 has no WARN option)
- Every Tier 2 rule is `PASS`, `NOT_APPLICABLE`, or `WARN` with concrete justification (no FAIL)
- Tier 3 rules are recorded but do not affect verdict

GUARD-CLEAN is required before `compact ship` will proceed.

---

## Tier 1 — BLOCKING Rules (silent break risk)

Violation produces immediate, hard-to-detect skill degradation. User cannot recover without code archaeology.

### QG-1: Source attribution preserved

CHANGES.md entry MUST include upstream SHA when the improvement copies or adapts external rules (caveman, agent-skills, or any other source).

**Why:** Drift detection. License compliance (MIT requires attribution).

**Evidence required:** CHANGES.md line/section anchor with `source: <repo>@<sha>` field.

**Auto-N/A:** No external source → cite "No external source" in evidence.

### QG-2: Document boundary not weakened

Improvement does NOT extend any output-tone overlay or compression mode (caveman, future tone modes, telegraphic styles, ultra-compression, etc.) into:
- Spec files (`specs/**/*.md`)
- Code files (`.ts`, `.tsx`, `.js`, etc.)
- Commit messages
- PR bodies
- Any disk-written artifact

**Why:** Document readability. Prevents AC degradation. Generalized beyond caveman to cover future tone modes.

**Evidence required:** SKILL.md `## Tone` section (or equivalent) cites the boundary explicitly.

### QG-3: Auto-clarity triggers preserved or expanded — never narrowed, never replaced silently

Every existing trigger string in the current trigger list MUST still match in the new set after improvement, OR appear in CHANGES.md as an explicit deprecation entry with replacement justification.

- Pure additions (new triggers added) → ALLOWED
- Replacements that semantically broaden but drop the literal old string → REQUIRES deprecation entry
- Removals → REQUIRES deprecation entry + alternative-protection plan

**Why:** Security floor. Once a trigger protects users, removing it (even via replacement-with-broader) = silent risk. Superset rule prevents replacement-narrowing.

**Evidence required:** Diff of old trigger set vs new trigger set. CHANGES.md deprecation entry if any literal trigger removed.

### QG-4: Action router phrase prefix backward compatible

Every existing `compact <cmd>` keyword (per current SKILL.md keyword list) still routes to its action after improvement. New keywords may be added; existing ones MUST NOT route to a different action.

**Why:** User muscle memory. Public contract.

**Evidence required:** Diff of SKILL.md frontmatter `description` keyword list AND Action Router section. No removal or rerouting of existing keywords.

**Auto-N/A:** No router edits → cite "No router changes".

### QG-5: Quality gates not bypassed (recursive guard)

Improvement does NOT add SKIP paths to lint, typecheck, build, test, or coverage gates beyond what `references/quality-gates.md` already permits; does NOT introduce silent skip flags.

**Recursive guard:** Any improvement that modifies `references/quality-gates.md` itself to permit additional SKIPs triggers a separate QG-5 audit on those additions — the rule does NOT self-justify gate-policy changes via the file it gates.

**Why:** Skill purpose = quality enforcement. Bypass = self-defeat. Recursive guard prevents Trojan-horse weakening of the gate policy itself.

**Evidence required:** Diff of `quality-gates.md`. Any SKIP additions = explicit QG-5 audit subsection in spec Notes citing why each addition is justified.

### QG-6: Spec template required sections preserved

Improvement does NOT remove User Journey, Acceptance Criteria, Codebase Impact, Scope, or Test Strategy from `references/spec-template.md`. May ADD sections.

**Why:** Spec rigor floor. Each section catches a class of bug.

**Evidence required:** Diff of spec-template.md. Mandatory sections (User Journey, ACs, Codebase Impact, Scope, Test Strategy) intact.

### QG-7: Loading mechanism validated

Improvement does NOT introduce file references that the action router cannot load (no `## Tone` style file pointers to non-action files). Inline what cannot be loaded.

**Why:** Caveman tone v1 had this bug. Silent failure mode where user thinks improvement active when it is not.

**Evidence required:** Every `references/...` link in SKILL.md or action files must point to a file loaded by the action router (action files OR files explicitly read by the loaded action). Tone overlays, mode switches, etc. MUST be inline in SKILL.md.

**Auto-N/A:** No new file references in SKILL.md → cite "No file references added".

### QG-8: Review coverage contract intact

`references/reviews/core-portable-review-spec.md` line coverage + rule coverage requirements unchanged or strengthened. Verdict vocabulary preserved (`PASS` / `WARN` / `FAIL` / `NOT_APPLICABLE`).

**Why:** Portable review spec = OSS asset. Weakening = breaking change for downstream consumers.

**Evidence required:** Diff of core-portable-review-spec.md. Coverage Contract section + Verdict Vocabulary section unchanged or strengthened only.

### QG-9: AC ↔ scope traceability enforced

Every Must Have AC in skill-improvement spec maps to ≥1 scope item; every scope item maps to ≥1 AC; no orphans either direction.

**Why:** Spec rigor. Orphan ACs = unverified work. Orphan scope = unjustified work.

**Evidence required:** Spec's Scope section uses `→ AC-N` mapping syntax. Bidirectional check passes.

### QG-10: Reversibility preserved

Improvement can be reverted without cascading damage. Single-commit revert restores prior behavior. No irreversible coupling between improvements.

**Why:** Fork hygiene. Allows hot-rollback if upstream pulls in a conflict.

**Evidence required:** Spec Notes documents revert plan: which commits, what survives, what is regenerated. Shipped specs are immutable; their orphan checklists after revert are acceptable.

---

## Tier 2 — WARN Rules (drift risk)

Violation produces gradual erosion or coordination failures over time. User can recover via subsequent revision. Justified `WARN` is acceptable; concrete justification required.

### QG-11: Override priority explicit

When improvement overlaps an existing rule, improvement spec states which wins + records in CHANGES.md anchor field.

**Why:** Avoids silent rule shadowing.

**Evidence required:** Spec Notes explicit override declaration. CHANGES.md anchor field documents the override.

### QG-12: Persistence clause for stateful overlays

Tone modes, level switches, multi-turn behaviors include explicit persistence semantics ("active until X" / "resets on Y").

**Why:** Prevents drift after many turns (caveman tone learned this).

**Evidence required:** SKILL.md (or appropriate file) documents persistence clause.

**Auto-N/A:** No stateful overlay → cite "No stateful behavior added".

### QG-13: Token efficiency neutral or positive

Improvement does NOT add verbose output where concise alternatives exist. Estimated token delta in spec Notes.

**Why:** compact-workflow brand promise.

**Evidence required:** Spec Notes "Token cost" subsection with estimated delta + justification if negative.

### QG-14: Failure hypotheses tested

Every HIGH or MEDIUM failure hypothesis in spec Analysis section has a corresponding test in Test Strategy (manual or automated).

**Why:** Spec Analysis section becomes hollow if hypotheses go untested.

**Evidence required:** Test Strategy section has hypothesis-level tests for each HIGH + MED.

### QG-15: Adversarial spec review evidence

Analysis section shows ≥3 perspectives ran (Skeptic + at least 2 others). Each blind spot has a fix or explicit ACCEPTED-RISK + reason.

**Why:** Spec rigor. Single perspective = blind spots ship.

**Evidence required:** Spec Analysis section enumerates perspectives. Blind Spots all resolved or marked ACCEPTED-RISK with reason.

### QG-16: CHANGES.md format compliance

Entry uses anchor + extractable fields per format defined in CHANGES.md header (post-bootstrap format).

**Why:** Method B migration readiness.

**Evidence required:** CHANGES.md entry follows format: `<file>: <type> | anchor: <marker> | extractable: yes|no | <reason>`.

### QG-17: Trigger overlap audit

New trigger phrases (action keywords, level switches) checked against existing compact-workflow keyword list AND Claude Code built-ins (`/compact`, `/review`, etc.).

**Why:** Prevents collision (caveman caught `/caveman` → `compact tone` swap for this reason).

**Evidence required:** Spec Notes "Trigger audit" subsection lists every new trigger + verifies no collision.

**Auto-N/A:** No new triggers → cite "No new triggers added".

### QG-18: Documentation drift check

README, SKILL.md, NOTICE.md, CHANGES.md, all `docs/*` files updated together when improvement changes user-facing behavior.

**Why:** Diverged docs = user confusion.

**Evidence required:** Spec Codebase Impact table includes every doc file affected.

---

## Tier 3 — INFO Rules (always informational)

Never enforced. Inform migration decisions + meta-consistency.

### QG-19: Improvement count tracked

CHANGES.md "Future improvements" count noted. If ≥10 improvements, surface Method B migration suggestion (per `docs/build-methods.md` crossover).

### QG-20: Self-application

Quality Guard rule itself is a skill-improvement → it must pass its own Quality Guard (proves the rule is internally consistent). Catches rules that are too vague to self-audit.

---

## Violation Report Format

When a Quality Guard violation occurs (during `compact spec`, `compact spec-review`, or `compact ship`), the report uses this format.

### Location

Inline in the spec under section `## Quality Guard Results` (appended by `compact spec` and re-rendered by `compact spec-review` / `compact ship`).

### Section structure

```markdown
## Quality Guard Results

*Last run: <ISO timestamp> — action: <plan|spec-review|ship> — quality-guard-version: v1.1*

| Rule | Tier | Verdict | Evidence | Notes |
|------|------|---------|----------|-------|
| QG-1 | 1 | PASS | CHANGES.md:42 | Source SHA recorded |
| QG-9 | 1 | FAIL | — | AC-E1 has no scope mapping |
| QG-13 | 2 | WARN | Notes section | Token cost trade-off accepted |
...

**Final verdict:** GUARD-CLEAN | NEEDS_FIXES (Tier 1 FAIL count: N) | NEEDS_FIXES (Tier 2 FAIL count: N)
```

### Agent narration prefix

When narrating a violation in chat output, the agent uses the marker `[QG-VIOLATION]` at the start of the line. This marker triggers auto-clarity (normal prose, no caveman compression) for the duration of the violation block. After the violation block, agent resumes normal tone behavior.

Example narration:

> `[QG-VIOLATION]` QG-9 FAIL: Acceptance criterion AC-E1 (grandfathering) maps to no scope item. Add a scope item explicitly handling grandfathered specs, or remove AC-E1 if redundant. Spec is BLOCKED until resolved.

This format is testable: the marker is a literal string, not a tone judgment.

---

## Action Integration

### `plan` action

When `compact spec` runs:
1. Generate spec per spec-template.md
2. After Codebase Impact section is written, run Detection Rule
3. If skill-improvement detected:
   a. Append `## Quality Guard Results` section using template above
   b. Pre-fill `NOT_APPLICABLE` per heuristic
   c. Mark all other rules as `[ ] PENDING` for user review
   d. Block dev-readiness until every required-tier rule resolved (PASS, N/A, or WARN with justification)

### `spec-review` action

When `compact spec-review` runs:
1. Detect skill-improvement classification (re-run Detection Rule)
2. If skill-improvement, add Quality Guard auditor as REQUIRED 4th perspective (in addition to Skeptic + 2 domain perspectives)
3. Auditor produces verdicts per rule; updates `## Quality Guard Results` section
4. Any Tier 1 FAIL OR Tier 2 FAIL = BLOCKING; spec-review cannot mark spec READY
5. Auditor narration uses `[QG-VIOLATION]` prefix marker

### `ship` action

When `compact ship` runs on skill-improvement spec:
1. Pre-iteration gate: re-read `## Quality Guard Results` section
2. If any Tier 1 or Tier 2 rule has flipped to FAIL since spec-review (e.g. spec edits since review broke a rule), refuse iteration 1
3. Report violation list using `[QG-VIOLATION]` prefix marker
4. User MUST resolve before ship proceeds

---

## Versioning

Quality Guard rules are versioned. Spec frontmatter pins `quality-guard-version: <current HEAD>` (e.g. `v1.1`) at spec creation. Future revisions to the Quality Guard:

1. Are themselves a skill-improvement spec (must pass Quality Guard at the version pinned in their own frontmatter)
2. Increment version (e.g. `v2`) on the rule set itself
3. Existing shipped specs retain their original pinned version (immutable)
4. New skill-improvement specs use the current HEAD version unless explicitly pinned

This avoids the bootstrap problem: a Quality Guard revision is audited against the rules it is changing — using the prior version, not the new version it is introducing.

---

## Out of Scope (v1)

- Automated CI lint of Quality Guard verdicts — manual only for v1
- Quality regression scoring (numeric quality delta per improvement) — qualitative pass/fail only
- Per-rule severity tunability — fixed Tier 1 / 2 / 3 in v1
- Integration with external policy engines (OPA, etc.)
- Contributor threat model + adversarial PR audit — future spec when repo accepts contributors
- Force-ship escape hatch — explicitly rejected (emergency-bypass is when quality slips most)

---

## Source

Defined by spec `specs/shipped/2026-04-30-skill-quality-guard.md`. Full failure-mode analysis + spec-review history + self-application audit recorded there.

---

## Changelog

### v1.1 (2026-04-30, post-ship corrigendum)

Driven by post-ship smoke audit (read-only audit ran by general-purpose agent against shipped v1). Two findings, both informational:

- **CHANGES.md-only friction** — Pre-fill heuristic now includes housekeeping shortcut: CHANGES.md-only specs auto-N/A QG-2..8 and QG-11..17, leaving 6 active rules (QG-1, QG-9, QG-10, QG-18, QG-19, QG-20). Reduces friction for trivial log-only specs without weakening the gate.
- **QG-7 evidence narration imprecision** — Audit caught wording in shipped spec QG-7 evidence ("loads on phrase match"); actual mechanism is "loads on detection match" (action file header conditional load when spec is classified skill-improvement). Shipped-spec narration corrected; rule text unchanged.

Per Quality Guard versioning rule, v1.1 is itself a skill-improvement → audited against pinned prior version v1. Corrigendum pattern: changes are additive (new pre-fill row) + narration-only (no rule semantics altered). No Tier 1 or Tier 2 rule wording changed; no SKIP added; no router edit. Logged in CHANGES.md.

### v1 (2026-04-30, original ship)

Initial Quality Guard. Spec: `specs/shipped/2026-04-30-skill-quality-guard.md`. Commit: `03eb1b5`.
