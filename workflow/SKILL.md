---
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
  forked_from: "5f6f937ad7e0bd02d8cbfbf4e3db6e460f10e469"
---

# Compact Workflow

High-velocity solo development. Idea to production same-day.

## Agent Capabilities

| Capability | Used For | Required | Fallback |
|------------|----------|----------|----------|
| File read/write | Specs, config, history | Yes | — |
| Code search (grep/glob) | Discovery, context | Yes | — |
| Shell/command execution | Quality gates (lint, build, test) | Yes | List commands for user to run |
| Codebase intelligence (`npx codebase-intelligence`) | Structural analysis for TS/TSX projects (graph, metrics, blast radius) | No | grep/glob/read (manual exploration) |
| Task/todo tracking | Phase management | Recommended | Track in spec Progress section |
| User interaction | Stuck escalation, risk flags | Recommended | Log decisions in spec Notes |
| Web/doc search | Pattern lookup | No | Use embedded patterns |

**Fallback rule:** If your agent lacks a capability, use the fallback. Never skip the workflow step — adapt the method.

## Commands

| Command | Action | Reference |
|---------|--------|-----------|
| `plan {idea}` | Create spec | [plan.md](references/actions/plan.md) |
| `spike {question}` | Time-boxed exploration | [spike.md](references/actions/spike.md) |
| `ship` / `ship {idea}` | Implement + validate | [ship.md](references/actions/ship.md) |
| `fix` / `fix {bug}` | Scientific debug + regression fix | [fix.md](references/actions/fix.md) |
| `review` | Portable multi-perspective review spec with line-by-line + rule-by-rule coverage | [review.md](references/actions/review.md) |
| `spec-review` | Adversarial spec analysis | [spec-review.md](references/actions/spec-review.md) |
| `focus` | Priority analysis + task proposals | [focus.md](references/actions/focus.md) |
| `done` | Validate + retro + archive | [done.md](references/actions/done.md) |
| `drop` | Abandon, preserve learnings | [drop.md](references/actions/drop.md) |
| `workflow` | Show state + suggest next | [Status](#status-action) (below) |

No flags needed. The agent auto-detects intent from context:
- "review the spec" → manual review pause
- "skip tests" → skip test gate (documented)
- "fix this bug" → dedicated bug fix with regression test
- "emergency fix" → bypass spec ceremony
- "production ready" → production validation

## Flow

```
Features: focus → plan {idea} → ship → [implement/review/fix loop] → done
Bug fixes: fix {bug} → [investigate/TDD/validate] → done
```

Quick mode (<2h): `ship {idea} → done`
Don't know what to work on: `focus`

## Philosophy

- **Spec-first**: All work needs a spec (creates one if missing)
- **Ship loop**: Build → review → fix until clean
- **Quality gates**: lint → typecheck → build → test → E2E → coverage (auto-detected per project)
- **E2E-first testing**: Default to E2E tests. Unit tests only for pure functions
- **TDD enforced**: RED → GREEN → REFACTOR per AC. Tests written before implementation (BLOCKING)
- **Mock boundary**: Real systems preferred. Mock only third-party APIs without sandbox (last resort)
- **AC-driven coverage**: Every Must Have + Error AC maps to an E2E test in the scenario registry
- **Anti-regression**: Bug fixes require E2E regression test + anti-cascade diff (BLOCKING)
- **Failure mode testing**: Every HIGH/MED failure hypothesis gets a defensive E2E test
- **Human controls deployment**: Agent codes, you push/deploy
- **Done same-day**: Scope to what ships today
- **Own planning**: Never use the host agent's built-in plan mode (EnterPlanMode, etc.). This skill writes real spec files to `specs/active/`.

## Spec Tiers

| Tier | Size | Spec | Task Tracking |
|------|------|------|---------------|
| trivial | <5 LOC | None — just do it | No |
| micro | <30 LOC | Inline comment in code | No |
| mini | <100 LOC | Spec file, minimal | Yes (if available) |
| standard | 100+ LOC | Full spec with checklist | Yes (if available) |

## Action Router

```
User input
  │
  ├─ "compact spec", "compact plan"                     → Load references/actions/plan.md
  ├─ "compact spike", "compact explore"                 → Load references/actions/spike.md
  ├─ "compact ship", "compact implement", "compact build" → Load references/actions/ship.md
  ├─ "compact fix", "compact debug", "compact repair"   → Load references/actions/fix.md
  ├─ "compact review"                                   → Load references/actions/review.md
  ├─ "compact spec-review", "compact challenge spec"    → Load references/actions/spec-review.md
  ├─ "compact focus", "compact prioritize"              → Load references/actions/focus.md
  ├─ "compact done", "compact finish"                   → Load references/actions/done.md
  ├─ "compact drop", "compact abandon"                  → Load references/actions/drop.md
  └─ "compact workflow", "compact status",
     "compact what's next", "compact-workflow"          → Status Action (below)
```

**Loading rule:** Read the action file BEFORE executing. The action file contains all logic, task templates, and references needed.

## Status Action

No separate action file — logic is inline here. Detect current state, suggest next action:

```
1. Check specs/active/ for active spec
2. Check git status for uncommitted work
3. Check task list for in-progress items

State → Suggestion:
  No spec, no changes    → "Ready. Run: compact spec {idea}"
  Active spec, no code   → "Spec ready. Run: compact ship"
  Active spec, code WIP  → "In progress. Run: compact ship (resumes)"
  Active spec, code done → "Ready to close. Run: compact done"
  No spec, dirty tree    → "Uncommitted work. Run: compact ship (creates spec) or compact done"
```

Output: Follow [status-output.md](references/templates/status-output.md).

## Project Structure

```
specs/
  active/       ← Current work (0-1 specs)
  backlog/      ← Queued work from focus
  shipped/      ← Completed features
  dropped/      ← Abandoned with learnings
  history.log   ← One-line per feature shipped/dropped
```

## Configuration

All behavior is configurable by editing the skill files directly.

| What to change | Edit |
|----------------|------|
| Action logic, gates, limits | `references/actions/{action}.md` |
| Output format | `references/templates/{action}-output.md` |
| Spec structure | `references/spec-template.md` |
| Quality gate commands/levels | `references/quality-gates.md` |
| Session resume, stuck detection | `references/session-management.md` |

## References

Actions:
- [Plan](references/actions/plan.md) | [Ship](references/actions/ship.md) | [Fix](references/actions/fix.md) | [Review](references/actions/review.md) | [Spec Review](references/actions/spec-review.md) | [Focus](references/actions/focus.md) | [Done](references/actions/done.md) | [Drop](references/actions/drop.md) | [Spike](references/actions/spike.md)

Output templates:
- [Plan + Spec Review](references/templates/plan-output.md) | [Ship](references/templates/ship-output.md) | [Fix](references/templates/fix-output.md) | [Review](references/templates/review-output.md) | [Focus](references/templates/focus-output.md) | [Done](references/templates/done-output.md) | [Drop](references/templates/drop-output.md) | [Spike](references/templates/spike-output.md) | [Status](references/templates/status-output.md)

Review standards:
- [Core portable review spec](references/reviews/core-portable-review-spec.md) | [Executor patterns](references/reviews/executor-patterns.md) | [Production Standards](references/reviews/production-standards.md)

The portable review spec is normative. Executor patterns are optional implementation guidance.

Specs & gates:
- [Spec template](references/spec-template.md) | [Quality gates](references/quality-gates.md) | [Session management](references/session-management.md) | [Memory update](references/memory-update.md) | [Testing automation](references/testing-automation.md) | [E2E scenarios](references/e2e-scenarios.md) | [Codebase intelligence](references/codebase-intelligence.md)

Patterns:
- [Implementation](references/patterns/implementation.md) | [Planning](references/patterns/planning.md) | [Debugging](references/patterns/debugging.md) | [Decisions](references/patterns/decisions.md) | [Decomposition](references/patterns/decomposition.md) | [Regression testing](references/patterns/regression-testing.md)

## Tone

CAVEMAN MODE ACTIVE — level: **full** (default). All compact-workflow conversational output (status updates, analysis, suggestions, retro lines, progress reports, plan summaries, ship iteration narration) uses caveman tone. Technical accuracy fully preserved.

**Override priority:** This `## Tone` section overrides prose style of every action file (`references/actions/*.md`) and every template (`references/templates/*.md`). Action files describe WHAT to output and structure; this section dictates HOW words are rendered. If an action template shows full sentences, render the same content in caveman tone.

### Persistence

Active every response. No revert after many turns. No filler drift. Still active if unsure. Off only on explicit `stop caveman` or `normal mode` from user. Re-applies on next compact-workflow action even if drift occurred.

### Rules

Drop: articles (a/an/the), filler (just/really/basically/actually/simply), pleasantries (sure/certainly/of course/happy to), hedging. Fragments OK. Short synonyms (big not extensive, fix not "implement a solution for"). Technical terms exact. Errors quoted exact.

Pattern: `[thing] [action] [reason]. [next step].`

Not: "Sure! I'd be happy to help. The issue is likely caused by..."
Yes: "Bug in auth middleware. Token expiry check use `<` not `<=`. Fix:"

### Levels

| Level | What change |
|-------|-------------|
| **lite** | Drop only filler + pleasantries. Articles kept. Full sentences. |
| **full** (default) | Drop articles, fragments OK, short synonyms. Classic caveman. |
| **ultra** | Maximum compression. Telegraphic. Technical density priority. |

Switch: user types `compact tone lite|full|ultra`. Default: full.

### Auto-Clarity (BOUNDARIES — write normal prose)

Caveman OFF for these blocks. Resume caveman after block done:

- **Security warnings** — exposed secrets, credential leaks, auth/authz holes, injection vectors, RLS gaps
- **Destructive ops confirmations** — `git push --force`, `git reset --hard`, `DROP TABLE`, `rm -rf`, `DELETE` without `WHERE`, branch deletion, force-push to main, irreversible DB ops, irreversible file deletion
- **Multi-step destructive sequences** — any sequence where fragment order risks misread of which step is destructive
- **User asks to clarify or repeats question** — fragment ambiguous; full sentence next turn
- **Code blocks, commit messages, PR bodies** — content unchanged (boundary already enforced by code/commit/PR rule below)

Trigger list authoritative — do not guess additional triggers. Add to spec if new trigger needed.

### Document boundary (CRITICAL)

Caveman applies to AGENT TURNS ONLY (chat output). Caveman does NOT apply to:

- **Spec files** (`specs/active/*.md`, `specs/backlog/*.md`, `specs/shipped/*.md`, `specs/dropped/*.md`) — full prose, articles present, normal sentences
- **Code files** — comments, docstrings, JSDoc unchanged
- **Commit messages** — Conventional Commits format, normal English
- **PR bodies** — normal prose, GitHub-rendered markdown
- **CHANGES.md, NOTICE.md, README.md** entries — normal prose
- **Any file the agent writes to disk** — caveman is output-tone only, never document-tone

If action template instructs "write to spec file" → switch to normal prose for that write → resume caveman in chat narration after.

### Source

Rules sourced from [`JuliusBrussee/caveman`](https://github.com/JuliusBrussee/caveman). Inline copy — no runtime dependency. Source SHA tracked in `CHANGES.md` and `references/tone/caveman.md`.

## Quality Guard

**Active for skill-improvement specs only — BLOCKING for dev-readiness and ship.**

Every spec that modifies the compact-workflow skill itself (paths under `workflow/`, root files `CHANGES.md` / `NOTICE.md` / `LICENSE`, or paths under `docs/`) is gated by the Quality Guard. Feature specs (about user code) are unaffected.

The Guard catches silent quality regressions:
- Source attribution drift (lost upstream SHA in CHANGES.md)
- Document boundary erosion (caveman or future tone modes leaking into spec files / code / commits)
- Auto-clarity trigger narrowing (security warnings losing protection)
- Action router corruption (existing `compact <cmd>` keywords broken)
- Quality gate bypass (silent SKIPs added)
- Loading mechanism breakage (file references that cannot be loaded)
- AC ↔ scope traceability loss
- Reversibility loss (improvements that cannot be cleanly reverted)

**Tiers:** 10 BLOCKING (Tier 1) + 8 WARN-eligible (Tier 2) + 2 informational (Tier 3). Mini-tier specs apply Tier 1 only; standard-tier specs apply Tier 1 + Tier 2.

**Version:** v1.1 (current HEAD; pinned in spec frontmatter as `quality-guard-version`). Spec revisions audit against pinned version, not mutable HEAD.

**No bypass:** No `--force` flag. No emergency override. Detection is deterministic; pre-fill heuristic reduces friction for trivially-N/A rules; auditor narration uses `[QG-VIOLATION]` prefix marker for normal-prose enforcement.

Full rules + detection + verdict rubric + violation report format: see [`references/quality-guard.md`](references/quality-guard.md).

Authoritative spec: `specs/shipped/2026-04-30-skill-quality-guard.md`.
