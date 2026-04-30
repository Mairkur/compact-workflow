---
title: Caveman tone overlay for compact-workflow
status: shipped
created: 2026-04-30
shipped: 2026-04-30
estimate: 1h
actual: 0.5h
tier: mini
caveman_source: JuliusBrussee/caveman@84cc3c14fa1e10182adaced856e003406ccd250d
---

# Caveman Tone Overlay

## Context

compact-workflow output is verbose by default (inherited from upstream workflow skill). Improvement: all action outputs use caveman-compressed tone — drop articles/filler, fragments OK, full technical accuracy preserved. ~65-75% token reduction on agent responses. Source rules from JuliusBrussee/caveman@main SKILL.md.

## Codebase Impact

| Area | Impact | Detail |
|------|--------|--------|
| `workflow/SKILL.md` | MODIFY (APPEND) | Add `## Tone` section at bottom — inline caveman rules directly (no file reference; no load-trigger mechanism exists for non-action files) |
| `workflow/references/tone/caveman.md` | CREATE | Documentation record only — source rules + SHA from JuliusBrussee/caveman; not loaded at runtime |
| `workflow/README.md` | MODIFY (APPEND) | Add caveat: caveman tone active by default |
| `CHANGES.md` | MODIFY (APPEND) | Log divergence entry with caveman source SHA |

**Files:** 1 create | 3 modify
**Reuse:** JuliusBrussee/caveman `skills/caveman/SKILL.md` — inline rules into SKILL.md `## Tone` section directly. Tone file = audit trail only.
**Breaking changes:** None — tone is output-only, no skill activation or router logic changes
**New dependencies:** None — rules copied inline, no runtime dependency on caveman repo

## User Journey

1. User types `compact spec add auth` → agent loads plan action → produces plan output in caveman tone (no "Sure! I'll help you...", no hedging, fragments, no articles) → user reads faster
2. User types `compact ship` → iteration updates terse ("Iter 1/3: auth middleware → DONE. lint ✓ typecheck ✓")
3. User types `compact done` → retro compressed ("Estimate: 2h → 1.5h. Worked: clean AC mapping. Missed: rate limiting.")
4. User types `compact status` → one-line state ("Active spec: auth. 2/3 scope done. Run: compact ship.")

Error: Security warning or destructive op in any output → **auto-clarity fires** → write full normal prose for that block → resume caveman after. No compressed security warnings.

Error: Tone not applied (agent drift) → output verbose → user types `compact status` again or starts next action → tone should re-apply per SKILL.md instruction persistence rule.

## Acceptance Criteria

- [ ] AC-1: GIVEN any compact-workflow action runs WHEN agent produces conversational output (status updates, analysis, suggestions, retro, progress) THEN response uses caveman tone (no articles, fragments OK, no filler/pleasantries, short synonyms) — tone overrides action file prose style
- [ ] AC-2: GIVEN caveman tone active WHEN output contains any of: `git push --force`, `DROP TABLE`, `rm -rf`, unrecoverable file deletion, exposed secret, irreversible DB op THEN that block written in full normal prose (auto-clarity) — resume caveman after
- [ ] AC-3: GIVEN caveman tone active WHEN output contains code block, commit message, or PR body THEN code/commit content unchanged (boundaries rule)
- [ ] AC-4: GIVEN caveman tone active WHEN agent writes content to spec file (`specs/active/*.md`) THEN spec file content written in normal prose (full sentences, articles present) — caveman applies to agent turns only, not documents
- [ ] AC-E1: GIVEN caveman tone applied WHEN technical content present (file paths, error messages, AC text, scope items) THEN technical accuracy preserved — no information dropped

## Scope

- [ ] 1. Append `## Tone` section to `workflow/SKILL.md` (inline caveman rules, not file reference) → AC-1, AC-2, AC-3, AC-4. Section must include: (a) full/lite/ultra rules inline, (b) explicit "overrides all action file prose style" clause, (c) auto-clarity trigger list, (d) spec-file boundary rule, (e) persistence clause
- [ ] 2. Create `workflow/references/tone/caveman.md` — documentation record only: source attribution, caveman SHA copied from, rules rationale. Not loaded at runtime. → (audit trail)
- [ ] 3. Append caveat to `workflow/README.md`: caveman tone active by default; how to disable → AC-1
- [ ] 4. Update `CHANGES.md` with divergence entry including `JuliusBrussee/caveman@<sha>` source reference → (housekeeping)

### Out of Scope

- Wenyan mode (Chinese classical) — not relevant for solo dev workflow
- Intensity level switching (`/caveman lite|ultra`) — future spec, default `full` only for now
- Per-action tone override — all actions use same tone
- Modifying upstream output template files — tone centralized in SKILL.md, not per template
- Automated tone regression testing — no test runner for skill files; manual only

## Quality Checklist

- [ ] All ACs passing (AC-1 through AC-4 + AC-E1)
- [ ] No regressions in existing trigger/router behavior
- [ ] Auto-clarity fires on defined trigger list (not ad-hoc guessing)
- [ ] Spec file content written normally — caveman not applied to `specs/active/*.md` content
- [ ] SKILL.md `## Tone` section is append-only at bottom — minimize upstream merge conflict surface
- [ ] CHANGES.md entry includes `JuliusBrussee/caveman@<sha>` for drift tracking
- [ ] README caveat present and accurate

## Test Strategy
Runner: none (skill files — no test runner) | E2E: manual trigger in fresh Claude Code session | TDD: write SKILL.md `## Tone` section → verify in fresh session before committing
AC-1 → manual: type `compact status` → verify fragments, no articles, no filler in response
AC-2 → manual: type `compact ship` on spec with a `git push --force` step → verify that step written in full prose, surrounding output caveman
AC-3 → manual: verify code blocks in `compact ship` output unchanged
AC-4 → manual: run `compact spec test` → open generated spec file → verify AC/scope/context sections written in full prose (not caveman)
AC-E1 → manual: check plan output contains all fields (spec path, tier, scope count, codebase impact) — none dropped
Mocks: none

## Analysis

*Spec review applied: 2026-04-30 — perspectives: Skeptic, DX Engineer, Integration Engineer*

**Assumptions Challenged:**

| Assumption | Evidence For | Evidence Against | Verdict |
|------------|-------------|-----------------|---------|
| SKILL.md tone instruction persists through action file loading | SKILL.md loaded first; agent carries session context | Action files use dense instructional prose; LLMs mirror register of text being read | RISKY → mitigated by "overrides all action file prose" clause in `## Tone` |
| Auto-clarity fires reliably on security warnings | Caveman repo includes rule | "Security warning" undefined — agent guesses boundary | RISKY → mitigated by explicit trigger list in AC-2 |
| File reference in SKILL.md loads tone overlay | — | No load-trigger mechanism exists for non-action files; action router only | WRONG → scope rewritten: inline rules in SKILL.md |
| Inline copy of caveman rules sufficient | MIT, static text | Caveman repo may update rules; copy drifts silently | RISKY → mitigated by caveman SHA in CHANGES.md |

**Blind Spots:**
1. **[loading]** `## Tone` file-reference approach fails silently — no error, no tone. Fixed: inline rules.
2. **[scope]** Spec file content written by agent during `compact spec` — caveman applied there = fragment ACs, no-article spec titles. Fixed: AC-4 + explicit boundary rule.
3. **[onboarding]** README silent on tone default. Fixed: scope item 3.
4. **[drift]** No caveman source SHA recorded. Fixed: scope item 4.

**Failure Hypotheses:**

| IF | THEN | BECAUSE | Severity | Mitigation |
|----|------|---------|----------|------------|
| Action files read after SKILL.md | Agent mirrors action file prose style | LLMs context-follow reading register | HIGH | "Overrides all action file prose style" clause |
| Inline rules omit "overrides" clause | Caveman applies to status but not plan/ship output | No explicit priority rule | HIGH | Clause required in scope item 1 |
| Spec content written in caveman | Spec files unreadable as documents | Agent doesn't switch modes mid-action | MED | AC-4 + spec-file boundary rule |

**The Real Question:** Confirmed — spec solves right problem. Word-level verbosity is target; structure handled by templates separately. Scope mechanism fix (inline not file-ref) critical for correctness.

**Open Items:**
- [improvement] Future spec: intensity level switching (`compact ship --lite`, etc.) → no action now
- [improvement] Future spec: per-action opt-out flag for structured report sharing → no action now
- [gap] No automated tone regression — known limit, accepted → no action

## Notes

## Timeline

| Action | Timestamp | Duration | Notes |
|--------|-----------|----------|-------|
| plan | 2026-04-30T12:40:00Z | - | Created |
| spec-review | 2026-04-30T12:50:00Z | - | 3 perspectives — loading mechanism rewritten, AC-4 added, auto-clarity triggers defined |
| ship | 2026-04-30T13:30:00Z | ~0.5h | All 4 scope items shipped: SKILL.md `## Tone` appended, references/tone/caveman.md created, README caveat appended, CHANGES.md entry finalized with caveman SHA `84cc3c14fa1e10182adaced856e003406ccd250d`. ACs verified: AC-1 (override clause), AC-2 (auto-clarity trigger list), AC-3 (code/commit/PR boundary), AC-4 (spec-file boundary), AC-E1 (technical accuracy). |
