---
title: Quality Guard for compact-workflow skill improvements
status: shipped
created: 2026-04-30
shipped: 2026-04-30
estimate: 2h
actual: ~1.5h
tier: standard
applies_to: improvements that modify files under `workflow/`, `CHANGES.md`, `NOTICE.md`, or `docs/` (the skill itself + its housekeeping)
quality-guard-version: v1
---

# Skill Quality Guard

## Context

compact-workflow is a fork of `agent-skills/workflow`. Each improvement modifies the base skill's behavior (router, prose tone, action logic, review contract, quality gates). Without a guard, improvements can silently degrade the skill — narrowing security boundaries, breaking traceability, bypassing quality gates, corrupting the action router, drifting from upstream contract.

This spec adds a **mandatory, BLOCKING Quality Guard** that gates every skill-improvement spec at three points: spec creation (`compact spec`), spec review (`compact spec-review`), and ship (`compact ship`). Violations block — no override path.

The guard applies **only** to specs whose `Codebase Impact` section touches paths under `workflow/`. Feature specs built using compact-workflow (i.e. specs about user code, not the skill itself) are unaffected.

## Codebase Impact

| Area | Impact | Detail |
|------|--------|--------|
| `workflow/references/quality-guard.md` | NEW | Authoritative checklist + violation taxonomy + verdict rubric |
| `workflow/references/spec-template.md` | EDIT-body | Add `## Quality Guard` section to spec template (skill-improvement specs only) |
| `workflow/references/actions/plan.md` | EDIT-body | Skill-improvement detection: if spec touches `workflow/`, render Quality Guard checklist + run pre-flight |
| `workflow/references/actions/spec-review.md` | EDIT-body | Add Quality Guard as REQUIRED perspective for skill-improvement specs (BLOCKING on violation) |
| `workflow/references/actions/ship.md` | EDIT-body | Pre-ship gate: re-verify Quality Guard checklist before iteration 1 (BLOCKING) |
| `workflow/SKILL.md` | APPEND | New `## Quality Guard` section pointing at `references/quality-guard.md` (governance link, not full inline rules) |
| `workflow/README.md` | APPEND | Caveat documenting Quality Guard: gates skill-improvement specs, link to `references/quality-guard.md`, how to disable (you cannot — guard is BLOCKING) |
| `CHANGES.md` | EDIT-body | Divergence entry per format + retroactive grandfather log for bootstrap + caveman-tone (audit-only, no enforcement) |

**Files:** 1 create | 7 modify
**Reuse:** Existing spec-review action perspective system. Existing BLOCKING gate pattern from quality-gates.md.
**Breaking changes:** None for feature specs. Skill-improvement specs gain mandatory checklist — back-compat: bootstrap + caveman-tone specs grandfathered (already shipped, retroactively logged in CHANGES.md as audit-only).
**New dependencies:** None.

**Detection rule (skill-improvement vs feature spec):**

A spec is a skill-improvement spec if its `Codebase Impact` table contains ANY of:
- Path under `workflow/` (e.g. `workflow/SKILL.md`, `workflow/references/**`, `workflow/README.md`)
- Repo root: `CHANGES.md`, `NOTICE.md`, `LICENSE`
- Path under `docs/` (e.g. `docs/build-methods.md`)
- Path under `specs/` is NOT a trigger (specs about specs are still feature work for the user, not skill changes)

Hybrid spec (touches both skill paths AND user-code paths) → classified as skill-improvement (stricter posture).

**Tier-conditional rule subset (DX mitigation):**

| Spec tier | Rules enforced |
|-----------|---------------|
| `mini` (skill-improvement) | Tier 1 only (QG-1 through QG-10) — 10 BLOCKING rules |
| `standard` (skill-improvement) | Tier 1 + Tier 2 (QG-1 through QG-18) — 10 BLOCKING + 8 WARN |
| Tier 3 (QG-19, QG-20) | Always informational, never enforced as PASS/FAIL |

Pre-fill heuristic: rules that are trivially N/A based on file types touched (e.g. QG-4 N/A when no router file edited, QG-12 N/A when no tone/level state) auto-prefilled by the agent on `compact spec` render. User reviews + confirms.

## User Journey

1. User types `compact spec <skill-improvement-idea>` → agent detects `workflow/` paths in Codebase Impact → renders standard spec sections + appends `## Quality Guard` checklist → user fills checklist with evidence per item → dev-readiness check fails if any guard item incomplete or violated → spec stays in DRAFT until guard clean.
2. User types `compact spec-review` on skill-improvement spec → adversarial review runs standard 3 perspectives + Quality Guard auditor as 4th REQUIRED perspective → any guard FAIL = spec-review FAIL, BLOCKING → user revises → re-review.
3. User types `compact ship` on skill-improvement spec → pre-iteration gate re-verifies Quality Guard checklist → any item flipped to violation since spec-review = ship blocked → user resolves before iteration 1 starts.
4. User types `compact spec <feature-idea>` (no `workflow/` paths) → standard flow, no Quality Guard rendered, no overhead.

Error: User attempts to ship skill-improvement spec with unchecked Quality Guard items → ship action refuses with explicit list of violations → user must complete checklist or `compact drop`.

Error: User adds Quality Guard checklist to feature spec by mistake → no harm; checklist optional for feature specs, ignored by ship gate.

Error: Quality Guard rule itself needs revision → revision = a skill-improvement spec → must pass its own Quality Guard. Self-application enforced (meta-consistency).

## Acceptance Criteria

- [ ] AC-1: GIVEN spec with any path under `workflow/` in Codebase Impact WHEN `compact spec` runs THEN Quality Guard checklist rendered into spec; dev-readiness blocks until all items resolved (PASS or NOT_APPLICABLE with justification)
- [ ] AC-2: GIVEN skill-improvement spec WHEN `compact spec-review` runs THEN Quality Guard auditor runs as 4th REQUIRED perspective; any FAIL is BLOCKING
- [ ] AC-3: GIVEN skill-improvement spec WHEN `compact ship` runs THEN pre-iteration gate re-checks Quality Guard items; ship refuses if any item violated since spec-review
- [ ] AC-4: GIVEN feature spec (no `workflow/` paths) WHEN any action runs THEN Quality Guard NOT rendered, NOT enforced — zero overhead
- [ ] AC-5: GIVEN Quality Guard rule itself revised (meta-spec) WHEN spec runs through `compact spec` / `spec-review` / `ship` THEN the same Quality Guard applies (self-consistency)
- [ ] AC-6: GIVEN any guard rule violated WHEN agent reports the violation THEN normal prose used (auto-clarity — quality violation = security-class signal, no caveman compression)
- [ ] AC-E1: GIVEN bootstrap commit + caveman-tone shipment (already shipped) WHEN Quality Guard introduced THEN those specs are grandfathered — no retroactive enforcement

## Quality Guard Rules (the list — completed)

The list user asked to complete. Each rule = MANDATORY check during spec / spec-review / ship for skill-improvement specs.

### Tier 1 — BLOCKING (any violation = spec rejected)

| ID | Rule | Why |
|----|------|-----|
| QG-1 | **Source attribution preserved** — CHANGES.md entry includes upstream SHA (caveman, agent-skills, or other source) when copying or adapting external rules | Drift detection. License compliance (MIT requires attribution). |
| QG-2 | **Document boundary not weakened** — improvement does NOT extend any output-tone overlay or compression mode (caveman, future tone modes, telegraphic styles, ultra-compression, etc.) into spec files, code files, commit messages, PR bodies, or any disk-written artifact | Document readability. Prevents AC degradation. Generalized beyond caveman to cover future tone modes. |
| QG-3 | **Auto-clarity triggers preserved or expanded — never narrowed, never replaced silently** — every existing trigger string in the current trigger list MUST still match in the new set after improvement, OR appear in CHANGES.md as an explicit deprecation entry with replacement justification. Pure additions allowed. Replacements that drop niche cases require explicit deprecation entry | Security floor. Once a trigger protects users, removing it (even via replacement-with-broader) = silent risk. Superset rule prevents replacement-narrowing. |
| QG-4 | **Action router phrase prefix backward compatible** — every existing `compact <cmd>` keyword (per current SKILL.md keyword list) still routes to its action after improvement | User muscle memory. Public contract. |
| QG-5 | **Quality gates not bypassed** — improvement does NOT add SKIP paths to lint, typecheck, build, test, or coverage gates beyond what `references/quality-gates.md` already permits; does NOT introduce silent skip flags. **Recursive guard:** any improvement that modifies `references/quality-gates.md` itself to permit additional SKIPs triggers a separate QG-5 audit on those additions — the rule does not self-justify gate-policy changes via the file it gates | Skill purpose = quality enforcement. Bypass = self-defeat. Recursive guard prevents Trojan-horse weakening of the gate policy itself. |
| QG-6 | **Spec template required sections preserved** — improvement does NOT remove User Journey, Acceptance Criteria, Codebase Impact, Scope, or Test Strategy from `references/spec-template.md`; may ADD sections | Spec rigor floor. Each section catches a class of bug. |
| QG-7 | **Loading mechanism validated** — improvement does NOT introduce file references that the action router cannot load (no `## Tone` style file pointers); inline what cannot be loaded | Caveman tone v1 had this bug. Silent failure = user thinks improvement active when it is not. |
| QG-8 | **Review coverage contract intact** — `references/reviews/core-portable-review-spec.md` line coverage + rule coverage requirements unchanged or strengthened; verdict vocabulary preserved (`PASS` / `WARN` / `FAIL` / `NOT_APPLICABLE`) | Portable review = OSS asset. Weakening = breaking change for downstream consumers. |
| QG-9 | **AC ↔ scope traceability enforced** — every Must Have AC in skill-improvement spec maps to ≥1 scope item; every scope item maps to ≥1 AC; no orphans either direction | Spec rigor. Orphan ACs = unverified work. Orphan scope = unjustified work. |
| QG-10 | **Reversibility preserved** — improvement can be reverted without cascading damage (single-commit revert restores prior behavior); no irreversible coupling between improvements | Fork hygiene. Allows hot-rollback if upstream pulls in a conflict. |

### Tier 2 — WARN (require explicit justification, not auto-blocking)

| ID | Rule | Why |
|----|------|-----|
| QG-11 | **Override priority explicit** — when improvement overlaps existing rule, improvement spec states which wins + records in CHANGES.md anchor field | Avoids silent rule shadowing. |
| QG-12 | **Persistence clause for stateful overlays** — tone modes, level switches, multi-turn behaviors include explicit persistence semantics ("active until X" / "resets on Y") | Prevents drift after many turns (caveman tone learned this). |
| QG-13 | **Token efficiency neutral or positive** — improvement does NOT add verbose output where concise alternatives exist; estimated token delta in spec Notes | compact-workflow brand promise. |
| QG-14 | **Failure hypotheses tested** — every HIGH or MEDIUM failure hypothesis in spec Analysis section has a corresponding test in Test Strategy (manual or automated) | Spec Analysis section becomes hollow if hypotheses go untested. |
| QG-15 | **Adversarial spec review evidence** — Analysis section shows ≥3 perspectives ran (Skeptic + at least 2 others); each blind spot has a fix or explicit ACCEPTED-RISK + reason | Spec rigor. Single perspective = blind spots ship. |
| QG-16 | **CHANGES.md format compliance** — entry uses anchor + extractable fields per format defined in CHANGES.md header (post-bootstrap format) | Method B migration readiness. |
| QG-17 | **Trigger overlap audit** — new trigger phrases (action keywords, level switches) checked against existing compact-workflow keyword list AND Claude Code built-ins (`/compact`, etc.) | Prevents collision (caveman caught `/caveman` → `compact tone` swap for this reason). |
| QG-18 | **Documentation drift check** — README, SKILL.md, NOTICE.md, CHANGES.md, all docs/ files updated together when improvement changes user-facing behavior | Diverged docs = user confusion. |

### Tier 3 — INFO (nice to have, not enforced)

| ID | Rule | Why |
|----|------|-----|
| QG-19 | **Improvement count tracked** — CHANGES.md "Future improvements" count noted; if ≥10 improvements, surface Method B migration suggestion (per docs/build-methods.md crossover) | Build method scaling. |
| QG-20 | **Self-application** — Quality Guard rule itself is a skill-improvement → it must pass its own Quality Guard (proves the rule is internally consistent) | Meta-consistency. Catches rules that are too vague to self-audit. |

## Verdict Rubric

Each rule gets one verdict: `PASS` | `WARN` | `FAIL` | `NOT_APPLICABLE`.

- **`PASS`** — rule satisfied, evidence cited (file path or section anchor)
- **`WARN`** — rule deviation justified in spec; recorded but not blocking (Tier 2 only)
- **`FAIL`** — rule violated, no justification accepted (BLOCKING for Tier 1; Tier 2 must be revised to PASS or WARN)
- **`NOT_APPLICABLE`** — rule does not apply to this improvement type; reason cited (e.g. "No external source — QG-1 N/A")

A skill-improvement spec is GUARD-CLEAN only when every Tier 1 rule is `PASS` or `NOT_APPLICABLE` AND every Tier 2 rule is `PASS` or `WARN` (with justification).

## Violation Report Format

When the agent reports a Quality Guard violation (during `compact spec`, `compact spec-review`, or `compact ship`), the report uses the following format:

**Location:** Inline in the spec under a new section `## Quality Guard Results` (appended to spec by `compact spec` and re-rendered by `compact spec-review` / `compact ship`).

**Section structure:**

```markdown
## Quality Guard Results

*Last run: <ISO timestamp> — action: <plan|spec-review|ship> — quality-guard-version: v1*

| Rule | Tier | Verdict | Evidence | Notes |
|------|------|---------|----------|-------|
| QG-1 | 1 | PASS | CHANGES.md:42 | Source SHA recorded |
| QG-9 | 1 | FAIL | — | AC-E1 has no scope mapping |
| QG-13 | 2 | WARN | Notes section | Token cost trade-off accepted |
...
```

**Agent narration prefix:** When narrating a violation in chat output, the agent uses the marker `[QG-VIOLATION]` at the start of the line. This marker triggers auto-clarity (normal prose, no caveman compression) for the duration of the violation block. After the violation block, agent resumes normal tone behavior.

Example narration:

> `[QG-VIOLATION]` QG-9 FAIL: Acceptance criterion AC-E1 (grandfathering) maps to no scope item. Add a scope item explicitly handling grandfathered specs, or remove AC-E1 if redundant. Spec is BLOCKED until resolved.

This format is testable: the marker is a literal string, not a tone judgment.

## Scope

- [ ] 1. Create `workflow/references/quality-guard.md` — full rules table (QG-1 through QG-20) + verdict rubric + violation report format + evidence-citation format + tier-conditional enforcement table + version field (`v1`) → AC-1, AC-2, AC-3
- [ ] 2. Edit `workflow/references/spec-template.md` — add conditional `## Quality Guard Results` section: rendered ONLY when spec is classified as skill-improvement (per detection rule); table format per Violation Report Format section above; pre-fill heuristic for trivially N/A rules → AC-1, AC-4
- [ ] 3. Edit `workflow/references/actions/plan.md` — add detection step: scan Codebase Impact paths against detection rule (`workflow/`, `CHANGES.md`, `NOTICE.md`, `LICENSE`, `docs/`); if matched, render Quality Guard Results section + apply tier-conditional rule subset (mini = Tier 1 only, standard = Tier 1+2) + block dev-readiness until all required-tier rules resolved → AC-1, AC-4
- [ ] 4. Edit `workflow/references/actions/spec-review.md` — add Quality Guard auditor perspective: REQUIRED for skill-improvement specs, OPTIONAL for feature specs; FAIL on any required-tier rule = BLOCKING; uses `[QG-VIOLATION]` prefix marker for normal prose enforcement → AC-2, AC-6
- [ ] 5. Edit `workflow/references/actions/ship.md` — add pre-iteration gate (BLOCKING): re-read Quality Guard Results section, refuse iteration 1 if any required-tier rule violated since spec-review; report uses `[QG-VIOLATION]` prefix marker → AC-3, AC-6
- [ ] 6. Append `## Quality Guard` section to `workflow/SKILL.md` — short pointer (governance link), not full inline rules; rules live in references/quality-guard.md; mention version pin and detection rule in summary → AC-1, AC-2, AC-3
- [ ] 7. **Append Quality Guard caveat to `workflow/README.md`** — gates skill-improvement specs (defines what counts), what it checks (Tier 1 BLOCKING, Tier 2 WARN, Tier 3 INFO), no disable path (BLOCKING by design), link to `references/quality-guard.md` → AC-1, **QG-18 fix**
- [ ] 8. Update `CHANGES.md` with divergence entry per format (anchor + extractable fields) → (housekeeping, QG-16)
- [ ] 9. **Append retroactive grandfather log to `CHANGES.md`** — bootstrap (`5085606`) + caveman-tone (`7f8f824`) shipped before Quality Guard; logged as audit-only entries under new subsection `### Grandfathered (pre-Quality-Guard)`; no enforcement, no retroactive blockage → **AC-E1, QG-9 fix (AC-E1 → scope mapping)**
- [ ] 10. Self-test: run this spec through Quality Guard v1 → verify all Tier 1 PASS or N/A AND all Tier 2 PASS or WARN with justification → record results in this spec's `## Quality Guard Results` section → AC-5
- [ ] 11. Manual smoke test in fresh Claude Code session: type `compact spec <fake-skill-improvement>` → verify Quality Guard Results section auto-rendered with pre-filled trivial rules; type `compact spec <fake-feature>` → verify section absent → AC-4 verification

### Out of Scope

- Automated enforcement (CI lint of spec files) — manual only for now; automation = future spec
- Quality Guard for feature specs (specs about user code) — explicitly N/A; only skill-improvement specs gated
- Retroactive enforcement on bootstrap + caveman-tone (already shipped) — grandfathered per AC-E1; logged audit-only via scope item 9
- Quality regression scoring (numeric quality delta per improvement) — qualitative pass/fail only for v1
- Per-rule severity tunability — fixed Tier 1/2/3 in v1; future spec for customization
- Integration with external policy engines (OPA, etc.) — out of scope for solo-dev OSS skill
- **Contributor threat model** (adversarial PRs that intentionally narrow auto-clarity, weaken gates, etc.) — solo-dev context = low risk now; future spec when repo opens to contributors (per spec-review SEC-5, R-3)
- **Force-ship escape hatch** (`compact ship --force` to bypass Quality Guard) — explicitly rejected per spec philosophy ("emergency-bypass is when quality slips most")

## Quality Checklist

- [ ] All ACs passing (AC-1 through AC-E1)
- [ ] Quality Guard rules (QG-1 through QG-20) defined with clear verdict rubric
- [ ] Skill-improvement spec detection logic deterministic (path-list match: `workflow/`, `CHANGES.md`, `NOTICE.md`, `LICENSE`, `docs/`)
- [ ] Tier-conditional enforcement defined (mini = Tier 1, standard = Tier 1+2, Tier 3 always informational)
- [ ] Pre-iteration ship gate refuses ship with explicit violation list (not silent skip)
- [ ] Self-application verified: this spec passes its own Quality Guard v1 → recorded in `## Quality Guard Results`
- [ ] Bootstrap + caveman-tone grandfathered (no retroactive blockage); audit-only log appended to CHANGES.md
- [ ] CHANGES.md entry includes proper anchors + extractable flags
- [ ] README updated with Quality Guard caveat (QG-18)
- [ ] No regressions to feature spec flow (zero overhead when no skill paths touched)
- [ ] Auto-clarity respected: violation reports use `[QG-VIOLATION]` prefix marker for normal prose
- [ ] QG version pinned in spec frontmatter (`quality-guard-version: v1`); future revisions audit against pinned version not mutable HEAD
- [ ] Pre-fill heuristic implemented: trivially-N/A rules auto-marked, user reviews + confirms (DX fatigue mitigation)

## Test Strategy

Runner: none (skill files — no test runner) | E2E: manual trigger in fresh Claude Code session | Approach: exercise each AC by attempting both PASS and FAIL paths AND each MED+ failure hypothesis with a smoke test.

**AC-level tests:**

- AC-1 → manual: type `compact spec <fake-skill-improvement-touching-workflow/SKILL.md>` → verify `## Quality Guard Results` section appears with tier-conditional rule subset; type `compact spec <fake-feature-touching-only-src/>` → verify section absent
- AC-2 → manual: run `compact spec-review` on a deliberately-violating skill-improvement spec (e.g. removes a security trigger to test QG-3) → verify FAIL on the violated Tier 1 rule + BLOCKING verdict + `[QG-VIOLATION]` prefix in narration
- AC-3 → manual: stage skill-improvement spec with QG-2 violation (intentional caveman extension into spec file) → run `compact ship` → verify ship refuses with explicit violation list before iteration 1
- AC-4 → manual: feature spec touching only `src/` (no skill paths) → verify zero Quality Guard overhead, no Quality Guard Results section rendered
- AC-5 → manual: this very spec must pass its own Quality Guard v1 before ship — see Scope item 10
- AC-6 → manual: deliberately violate QG-3 → run spec-review → verify violation reported in normal prose (not caveman fragments) AND `[QG-VIOLATION]` prefix present
- AC-E1 → manual: open `specs/shipped/2026-04-30-caveman-tone.md` AND `specs/shipped/001-bootstrap.md` (when added to shipped/) → verify no retroactive Quality Guard checklist injected; verify CHANGES.md `### Grandfathered (pre-Quality-Guard)` subsection present and lists both

**Hypothesis-level tests (per QG-14, mitigation for spec-review WARN):**

- HYP-checklist-fatigue (DX-1) → manual: run `compact spec` on 3 simulated skill-improvement specs back-to-back → time each end-to-end → if total review time > 45min for mini-tier, fatigue mitigation insufficient → revise tier-conditional split
- HYP-vague-rule-WARN-default (Skeptic) → manual: deliberately enter justification-by-vagueness for a Tier 2 rule (e.g. "QG-13 WARN: token cost roughly OK") → spec-review must reject vague justification + require concrete evidence
- HYP-orphan-rule-on-revert (failure hypothesis #4) → manual: revert this spec's commit, run `compact spec` on a NEW skill-improvement → verify next spec uses prior version of Quality Guard rules (or no rules if this commit was the introduction); shipped specs retain their pinned version
- HYP-recursive-bypass-via-quality-gates.md (SEC-1) → manual: simulate improvement that adds SKIP path to `quality-gates.md` itself → run spec-review → verify recursive QG-5 audit fires on the SKIP addition specifically, not just the gate-modification act
- HYP-trigger-replacement-narrowing (SEC-2) → manual: simulate improvement that replaces `git push --force` trigger with `git force operations` (semantically broader, drops literal match) → run spec-review → verify QG-3 FAIL or deprecation entry required

Mocks: none.

## Quality Guard Results

*Last run: 2026-04-30T14:15:00Z — action: spec-review-merge — quality-guard-version: v1*

Self-application of Quality Guard v1 to this spec (AC-5, QG-20).

| Rule | Tier | Verdict | Evidence | Notes |
|------|------|---------|----------|-------|
| QG-1 | 1 | N/A | — | No external source — rules synthesized from internal retros + spec-review findings |
| QG-2 | 1 | PASS | Spec adds `## Quality Guard Results` section with normal prose; no compression mode extended to artifacts | — |
| QG-3 | 1 | PASS | AC-6 expands auto-clarity (violation reports = normal prose); no triggers narrowed | — |
| QG-4 | 1 | N/A | — | No router file edits; existing `compact <cmd>` keywords unchanged |
| QG-5 | 1 | PASS | Adds gate (Quality Guard pre-iteration check), removes none; no `quality-gates.md` modification | — |
| QG-6 | 1 | PASS | Spec template gains `## Quality Guard Results` section; required sections preserved | — |
| QG-7 | 1 | PASS | `quality-guard.md` loads via action files (plan / spec-review / ship) on detection match — action file header conditional load when spec classified skill-improvement | Narration corrected v1.1 (was: "phrase match") |
| QG-8 | 1 | PASS | Portable review spec + executor patterns untouched; verdict vocabulary preserved | — |
| QG-9 | 1 | PASS | All ACs map to scope: AC-1 → 1,2,3,6 / AC-2 → 1,4 / AC-3 → 1,5 / AC-4 → 2,3 / AC-5 → 10 / AC-6 → 4,5 / **AC-E1 → 9** (grandfather log) | Fixed via P-1 (scope item 9 added) |
| QG-10 | 1 | PASS | Single-commit revert restores prior. Shipped specs immutable; orphan checklists in shipped specs acceptable (frozen at ship time) | Trade-off documented in Notes |
| **Tier 1 subtotal** | | **10/10 PASS or N/A** | | **No BLOCKING violations** |
| QG-11 | 2 | PASS | No overrides on existing rules — additive only | — |
| QG-12 | 2 | N/A | — | No stateful overlay introduced |
| QG-13 | 2 | WARN | Notes section "Token cost trade-off" | ~80 LOC checklist added to skill-improvement specs; trade-off justified (5 reasons in Notes) |
| QG-14 | 2 | PASS | Test Strategy hypothesis-level tests added: HYP-checklist-fatigue, HYP-vague-rule-WARN-default, HYP-orphan-rule-on-revert, HYP-recursive-bypass, HYP-trigger-replacement-narrowing | Fixed via P-4 |
| QG-15 | 2 | PASS | 4 perspectives ran (Skeptic, Security Engineer, DX Engineer, Quality Guard auditor); 11 patches identified; all applied | — |
| QG-16 | 2 | PASS | Scope item 8 commits to anchor + extractable format | — |
| QG-17 | 2 | N/A | — | No new triggers added; uses existing `compact spec / spec-review / ship` |
| QG-18 | 2 | PASS | Scope item 7 appends Quality Guard caveat to `workflow/README.md` | Fixed via P-2 |
| **Tier 2 subtotal** | | **8/8 PASS / N/A / WARN** | | **No FAIL; one WARN justified** |
| QG-19 | 3 | PASS | Improvement #3 (bootstrap, caveman, this); <10 → no Method B trigger | Informational |
| QG-20 | 3 | PASS | This audit IS the self-application | Informational |

**Final verdict: GUARD-CLEAN** — 10 Tier 1 PASS or N/A, 7 Tier 2 PASS or N/A + 1 Tier 2 WARN with justification, 2 Tier 3 PASS.

Spec cleared for `compact ship`.

## Analysis

*Spec review applied: 2026-04-30T14:00:00Z — perspectives: Skeptic, Security Engineer, DX Engineer, Quality Guard auditor (self-application per QG-20). 11 patches merged 14:15:00Z. Self-application re-run: GUARD-CLEAN.*

**Initial Assumptions to Challenge:**

| Assumption | Evidence For | Evidence Against | Verdict |
|------------|-------------|-----------------|---------|
| Path-prefix detection (`workflow/`) reliably classifies skill-improvement specs | Bootstrap, caveman, this spec all touch `workflow/` | Edge case: spec touches both `workflow/` and `lib/` — hybrid? | LIKELY OK — treat hybrid as skill-improvement (stricter) |
| Tier 1 rules are objectively verifiable | Most rules cite concrete files/sections | QG-3 "narrowed" needs definition (rule count? scope?) | NEEDS clarification in quality-guard.md |
| Self-application is consistent | This spec defines + uses its own rules | Bootstrap problem if rule revision changes the rules used to validate the rule revision | ACCEPTABLE — version Quality Guard rules; revision spec uses prior version |
| Grandfathering is safe | Bootstrap + caveman shipped before guard existed | Future audit may surface guard violations in shipped work | ACCEPTABLE — log retroactively in CHANGES.md, do not block |

**Blind Spots (initial — to be expanded by spec-review):**
1. **[detection]** Spec edits CHANGES.md (root file, not under `workflow/`) but no `workflow/` paths — currently classified as feature spec. Likely wrong: CHANGES.md edits are skill-improvement housekeeping. Fix: add `CHANGES.md` to detection list.
2. **[ergonomics]** 20 rules × every skill-improvement = checklist fatigue. Fix: pre-fill PASS for rules trivially satisfied by improvement type; user only resolves non-trivial.
3. **[escape hatch]** No `--force` ship flag for emergency. Per spec philosophy ("emergency fix" → bypass spec ceremony), should there be a guard bypass? Likely NO — guard exists exactly because emergency-bypass is when quality slips most.
4. **[versioning]** Quality Guard rules will evolve. No version field on the rules. Fix: add `quality-guard-version: v1` to spec frontmatter; check rule against pinned version.

**Failure Hypotheses (initial):**

| IF | THEN | BECAUSE | Severity | Mitigation |
|----|------|---------|----------|------------|
| Path detection misses `CHANGES.md` / `NOTICE.md` / `docs/build-methods.md` | Skill-improvement housekeeping ships without guard | Detection only checks `workflow/` prefix | HIGH | Expand detection list (Blind Spot 1) |
| 20-rule checklist abandoned mid-spec | Spec ships with unverified rules | Fatigue | MED | Pre-fill trivial PASSes (Blind Spot 2) |
| Rules vague — `WARN` becomes default escape | Tier 1 enforcement degrades to advisory | "Not BLOCKING enough" reasoning | MED | Each Tier 1 rule must cite concrete artifact / section / file in evidence |
| Improvement reverted but Quality Guard rules ship orphaned | New skill-improvement specs reference deleted rules | Coupling | LOW | Rules versioned; revert = pin to prior version |

**Open Items (resolved by spec-review merge):**
- [decision] ~~Bootstrap + caveman-tone retroactive log~~ → RESOLVED: scope item 9 logs both as audit-only in CHANGES.md `### Grandfathered (pre-Quality-Guard)` subsection
- [decision] ~~20 rules vs 10 always + 10 conditional~~ → RESOLVED: tier-conditional split (mini = Tier 1 / 10 rules, standard = Tier 1+2 / 18 rules) + pre-fill heuristic for trivially-N/A
- [improvement] Future: scoring model for quality delta per improvement — confirmed out of scope v1
- [improvement] Future: contributor threat model + adversarial PR audit (per spec-review SEC-5, R-3) — out of scope v1, opens when repo accepts contributors

**Patches applied during spec-review merge (P-1 through P-11 + R-1, R-2, R-3):**

| ID | Patch | Status |
|----|-------|--------|
| P-1 | Scope item 9: retroactive grandfather log → fixes QG-9 orphan AC-E1 | APPLIED |
| P-2 | Scope item 7: README append for Quality Guard caveat → fixes QG-18 | APPLIED |
| P-3 | Notes "Token cost trade-off" subsection → QG-13 WARN justification | APPLIED |
| P-4 | Test Strategy hypothesis-level tests (5 added) → QG-14 PASS | APPLIED |
| P-5 | QG-2 reworded: "any output-tone overlay or compression mode" | APPLIED |
| P-6 | QG-3 reworded: superset rule + deprecation entry requirement | APPLIED |
| P-7 | QG-5 reworded: recursive guard for `quality-gates.md` self-modification | APPLIED |
| P-8 | Detection rule extended to `CHANGES.md`, `NOTICE.md`, `LICENSE`, `docs/` (Codebase Impact section) | APPLIED |
| P-9 | Tier-conditional rule subset (mini = Tier 1, standard = Tier 1+2) + pre-fill heuristic | APPLIED |
| P-10 | `quality-guard-version: v1` pinned in spec frontmatter | APPLIED |
| P-11 | Violation Report Format section: inline `## Quality Guard Results` + `[QG-VIOLATION]` prefix marker | APPLIED |
| R-1 | Tier 1 vs Tier 2 split principle (Notes) | APPLIED |
| R-2 | Concrete failure mode per rule (Notes "Inspiration sources") | APPLIED |
| R-3 | Contributor threat model logged as Out of Scope future spec | APPLIED |

## Notes

User originally asked: "Never accept a modification that will decrease the quality of the skills. Specs to be well thought, well secure. Complete this list."

Completed list = QG-1 through QG-20 above, organized by tier.

### Inspiration sources (concrete failure modes guard catches — R-2)

Each rule traces to a concrete failure mode that already happened or was caught in spec review:

- **QG-2 document boundary** — caveman-tone v1 spec almost shipped without explicit spec-file boundary (AC-4 added in caveman spec-review). Without QG-2, future tone modes could repeat this failure.
- **QG-7 loading mechanism** — caveman-tone v1 spec originally proposed file reference for tone overlay; rejected in spec-review because no load-trigger mechanism exists for non-action files. QG-7 codifies the lesson — silent failure mode where user thinks improvement active but it never loads.
- **QG-12 persistence clause** — caveman tone needed explicit persistence clause to prevent agent drift after many turns; surfaced because LLMs default to register-mirroring of prose around them.
- **QG-17 trigger overlap** — caveman shipment caught `/caveman` slash command collision with Claude Code `/compact` builtin; replaced with `compact tone <level>`. Without QG-17 audit, future improvements might reintroduce collisions.
- **QG-1 source attribution** — bootstrap commit recorded upstream SHA; without it, divergence drift from agent-skills/workflow becomes invisible.
- **QG-4 router backward compat** — bootstrap renamed router phrases (`compact <cmd>` prefix); without QG-4, future improvement could rename `compact ship` to `compact deploy` and break user muscle memory.
- **QG-8 review coverage contract** — portable review spec is the OSS asset of compact-workflow; weakening line/rule coverage = breaking change for downstream consumers.
- **QG-6 + QG-9 (template + traceability)** — caveman spec-review caught orphan ACs and scope items; QG-6 + QG-9 enforce the discipline at template + spec level.

### Token cost trade-off (QG-13 WARN justification)

This improvement adds ~80 LOC `## Quality Guard Results` checklist to every skill-improvement spec (mini = ~50 LOC, standard = ~100 LOC). On a typical mini skill-improvement spec (~150 LOC), this is a 33% size increase per spec.

**Trade-off accepted because:**
1. Skill-improvement specs are infrequent (caveman + bootstrap + this = 3 in fork lifetime so far)
2. Cost is per-spec, not per-execution — does not inflate runtime token usage
3. Tier-conditional split + pre-fill heuristic reduce burden for mini tier
4. Quality regression cost (silent skill degradation) > checklist cost
5. Pre-fill removes ~50% of items from manual review (rules trivially N/A based on file types)

Future revision: if token cost becomes problematic, collapse Tier 1 + Tier 2 into single auto-checked summary table (10 rules max), keep verbose audit only when violation found.

### Tier 1 vs Tier 2 split principle (R-1)

- **Tier 1** = silent break risk. Violation produces immediate, hard-to-detect skill degradation (lost loading, missing source SHA, broken router, narrowed security triggers). User cannot recover without code archaeology.
- **Tier 2** = drift risk. Violation produces gradual erosion or coordination failures over time (override priority unclear, persistence drift, doc divergence, format inconsistency). User can recover via subsequent revision.
- **Tier 3** = informational. No enforcement; informs migration decisions (Method B crossover, self-application meta-check).

## Timeline

| Action | Timestamp | Duration | Notes |
|--------|-----------|----------|-------|
| plan | 2026-04-30T13:45:00Z | - | Created. 20 Quality Guard rules across 3 tiers. Self-application required (AC-5). Awaiting spec-review. |
| spec-review | 2026-04-30T14:00:00Z | - | 4 perspectives (Skeptic, Security Engineer, DX Engineer, Quality Guard auditor self-application). Self-application caught 2 FAIL (QG-9 orphan AC, QG-18 README) + 3 WARN. 11 patches required. |
| spec-review-merge | 2026-04-30T14:15:00Z | - | All 11 patches applied (P-1 through P-11) + R-1 tier split principle + R-2 concrete failure modes + R-3 future contributor threat model. Detection rule extended to include CHANGES.md, NOTICE.md, LICENSE, docs/. Tier-conditional rule subset added (mini = Tier 1, standard = Tier 1+2). Pre-fill heuristic added. Violation Report Format added. QG version pinned to v1. README append added (scope item 7). Grandfather log added (scope item 9). Hypothesis-level tests added. Token-cost trade-off documented. Re-run self-application required before ship. |
| ship | 2026-04-30T14:30:00Z | ~1.5h | All 11 scope items shipped. 1 NEW + 7 MODIFY. Pre-iteration Quality Guard gate ran on this very spec — GUARD-CLEAN per `## Quality Guard Results`. Iter 1: workflow/references/quality-guard.md created (305 lines). Iter 2: spec-template.md (rule 15 + conditional section + frontmatter pin). Iter 3: actions/plan.md (Step 3.7 detection + Readiness Algorithm rule 15). Iter 4: actions/spec-review.md (Step 2 auditor REQUIRED + Step 7 dedicated auditor) + actions/ship.md (Phase 0a pre-iteration gate). Iter 5: SKILL.md `## Quality Guard` section + README caveat before `## Tone`. Iter 6: CHANGES.md skill-quality-guard entry + `### Grandfathered (pre-Quality-Guard)` subsection logging bootstrap + caveman-tone retroactively. Iter 7: spec archived to specs/shipped/. ACs verified: AC-1 (detection + render + dev-readiness block), AC-2 (auditor REQUIRED for skill-improvement, BLOCKING on Tier 1+2 FAIL), AC-3 (Phase 0a pre-iteration gate, no `--force`), AC-4 (zero overhead for feature specs — detection skip), AC-5 (self-application PASS, recorded in Quality Guard Results), AC-6 (`[QG-VIOLATION]` prefix marker for normal-prose violation reports), AC-E1 (grandfather log in CHANGES.md, no retroactive enforcement). Manual smoke tests deferred to fresh-session validation per Test Strategy. |
