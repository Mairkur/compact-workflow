# compact-workflow — High-Velocity Solo Development

Idea to production same-day. Spec-first, quality-gated, pattern-driven.

Fork of [agent-skills/workflow](https://github.com/bntvllnt/agent-skills). Invoked via `compact <subcommand>` prefix to coexist with the upstream skill.

> **Note:** `/compact` is a Claude Code built-in command (transcript compression). Do not type `/compact` to invoke this skill — use `compact <subcommand>` (e.g. `compact spec`, `compact ship`).

## Install

```bash
ln -s ~/Documents/GitHub/compact-workflow/workflow ~/.claude/skills/compact-workflow
```

Verify:

```bash
ls ~/.claude/skills/compact-workflow/SKILL.md
```

## Commands

| Command | What |
|---------|------|
| `compact spec {idea}` | Create spec with user journey, ACs, scope |
| `compact spike {question}` | Time-boxed exploration, produces go/no-go |
| `compact ship` or `compact ship {idea}` | Implement with build/review/fix loop |
| `compact review` | Portable multi-perspective review spec with line-by-line + rule-by-rule coverage |
| `compact focus` | Scan codebase, prioritize tasks toward production readiness |
| `compact done` | Validate, retro, archive, propose memory update |
| `compact drop` | Abandon with learnings preserved |
| `compact workflow` | Show current state, suggest next action |

No flags needed — the agent auto-detects intent from your natural language (e.g., "compact review the spec", "compact ship — skip tests", "compact ship — emergency fix").

## Portable review standard

The `compact review` command is defined as a **core portable review spec**:
- the core contract defines what review must cover and what evidence it must emit
- executors are optional implementation patterns that can be single-agent, multi-agent, CI, or human/manual
- the compact-workflow skill stays runtime-neutral in open source

See:
- `references/reviews/core-portable-review-spec.md`
- `references/reviews/executor-patterns.md`

## Quickstart

Don't know what to work on? Start with focus:

```
You:   compact focus

Agent: Scanning codebase against 9 review perspectives...
       Goal: Production readiness

       Perspective Summary:
         Security:    2 gaps (HIGH)
         Testability: 3 gaps (MEDIUM)
         Reliability: 1 gap (MEDIUM)

       Do First:
         1. Add input validation to /api/auth endpoints (Security, ~30min)
         2. Add error handling to database queries (Reliability, ~1h)

       Quick Wins:
         3. Add missing test for user creation flow (Testability, ~30min)

       Select tasks → specs created in specs/backlog/
```

Here's what a real workflow looks like end-to-end:

```
You:   compact spec add JWT authentication

Agent: Parsing idea...
       Tier: mini (estimated ~80 LOC)

       Writing user journey...
         ACTOR: Authenticated user
         GOAL: Log in with email/password, receive JWT
         1. User submits credentials → System validates → User receives token
         2. User sends request with token → System verifies → User gets response
         E1. Invalid credentials → System returns 401 → User sees error message

       Writing acceptance criteria...
         AC-1: GIVEN valid credentials WHEN POST /auth/login THEN returns JWT token
         AC-2: GIVEN valid JWT WHEN GET /api/protected THEN returns 200
         AC-E1: GIVEN invalid credentials WHEN POST /auth/login THEN returns 401

       Mapping scope to ACs...
         1. Create auth middleware → AC-2
         2. Create login endpoint → AC-1, AC-E1
         3. Add tests → AC-1, AC-2, AC-E1

       Running dev-readiness check...
         User journey:       PASS
         Acceptance criteria: PASS
         Scope ↔ AC:         PASS (3 scope → 3 ACs, 0 orphans)
         Quality checklist:   PASS

       Dev-ready: READY
       Spec: specs/active/2025-01-29-jwt-auth.md

       Next: Run `compact ship` to implement.

You:   compact ship

Agent: Loading spec... mode: LOOP (3 scope items, max 5 iterations)

       Iteration 1: Implementing scope item 1 (auth middleware → AC-2)
         ✓ Created src/middleware/auth.ts
         Quick pass: lint ✓ typecheck ✓
         Scope item 1 [x]

       Iteration 2: Implementing scope item 2 (login endpoint → AC-1, AC-E1)
         ✓ Created src/routes/auth.ts
         Quick pass: lint ✓ typecheck ✓
         Scope item 2 [x]
         Running dev review (5 core perspectives + Rules when config files exist)...
           Correctness: PASS
           Security: WARN — add rate limiting to login (medium)
           Reliability: PASS
           Performance: PASS
           DX: PASS

       Iteration 3: Implementing scope item 3 (tests → AC-1, AC-2, AC-E1)
         ✓ Created src/__tests__/auth.test.ts
         Quick pass: lint ✓ typecheck ✓
         Scope item 3 [x]

       Running full pass...
         Lint: ✓  Typecheck: ✓  Build: ✓  Tests: 3/3 passing ✓

       Exit check:
         Must Have ACs: 2/2 ✓  Error ACs: 1/1 ✓  Scope: 3/3 ✓
         Gates: green  Blockers: 0

       CLEAN — Ready. Run `compact done`.

You:   compact done

Agent: Validating...
         All Must Have ACs [x] ✓
         All Error ACs [x] ✓
         New tests for new behavior ✓ (3 tests added)
         Existing tests passing ✓
         No lint/typecheck errors ✓
         No hardcoded secrets ✓

       Retro:
         Estimate: 2h → Actual: 1.5h (75%)
         Worked: Clean scope-to-AC mapping, no orphan work
         Didn't: Missed rate limiting — caught in review
         Next time: Add security checklist for auth features

       Proposing memory update...
         "Auth features should include rate limiting in quality checklist"
         → Update project agent config? [yes/no]

       Archived: specs/shipped/2025-01-29-jwt-auth.md

       Human actions: commit → push → deploy → verify
```

## User Journey: Spec to Production

```
┌─────────────────────────────────────────────────────────────────────┐
│                        SPEC TO PRODUCTION                          │
│                                                                     │
│  YOU                           AGENT                     OUTPUT     │
│  ───                           ─────                     ──────     │
│                                                                     │
│  "compact spec add rate limiting"                                   │
│       │                                                             │
│       ├──────────────────▶  Parse idea                              │
│                             Select tier: mini                       │
│                             Write user journey:                     │
│                               ACTOR: API consumer                   │
│                               GOAL: Protected from abuse            │
│                               1. Client sends request               │
│                               2. System checks rate                 │
│                               3. Under limit → process              │
│                               E1. Over limit → 429 + Retry-After   │
│                             Write ACs (GIVEN/WHEN/THEN)             │
│                             Map scope ↔ ACs                         │
│                             Quality checklist                       │
│                             Dev-readiness: READY ──────▶ spec file  │
│                                                                     │
│  "compact ship"                                                     │
│       │                                                             │
│       ├──────────────────▶  Load spec, create tasks                 │
│                             Iteration 1: middleware ──▶ lint ✓      │
│                             Iteration 2: config ──────▶ lint ✓      │
│                                Dev review (5 core + Rules)          │
│                             Iteration 3: tests ───────▶ lint ✓      │
│                             Full pass: lint ✓ types ✓               │
│                                         build ✓ test ✓              │
│                             Exit: all ACs ✓ ──────────▶ CLEAN      │
│                                                                     │
│  "also add per-user limits"   (MID-LOOP CHANGE)                    │
│       │                                                             │
│       ├──────────────────▶  PAUSE implementation                    │
│                             Update spec:                            │
│                               + scope item 4: per-user config      │
│                               + AC-4: GIVEN user config WHEN...     │
│                               + journey step 4                      │
│                             Re-check traceability                   │
│                             Update tasks                            │
│                             RESUME loop ───────────────▶ continues  │
│                                                                     │
│  "compact done"                                                     │
│       │                                                             │
│       ├──────────────────▶  Validate all ACs ✓                     │
│                             New tests exist ✓                       │
│                             No lint/type errors ✓                   │
│                             Retro captured                          │
│                             Memory update proposed                  │
│                             Spec archived ─────────────▶ shipped/   │
│                                                                     │
│  YOU (manual)                                                       │
│       │                                                             │
│       ├─ git commit + push                                          │
│       ├─ deploy                                                     │
│       └─ verify in production                                       │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

Key: The spec is updated at every stage. Mid-loop changes are captured in the spec BEFORE implementation. The spec is always the source of truth.

## Reading Order

```
Start here
    │
    ▼
┌──────────┐     Already know           ┌──────────────────────┐
│ README   │──── the basics? ──────────▶│ SKILL.md             │
│ (you are │                            │ (agent instructions)  │
│  here)   │                            └──────────────────────┘
└────┬─────┘
     │ Want to understand a phase?
     │
     ├── Focus:   references/actions/focus.md
     ├── Plan:    references/actions/plan.md
     ├── Spike:   references/actions/spike.md
     ├── Ship:    references/actions/ship.md
     ├── Review:  references/actions/review.md
     ├── Done:    references/actions/done.md
     ├── Drop:    references/actions/drop.md
     └── Memory:  references/memory-update.md

     │ Need a pattern?
     │
     ├── Building: references/patterns/implementation.md
     ├── Planning: references/patterns/planning.md
     ├── Debugging: references/patterns/debugging.md
     ├── Deciding: references/patterns/decisions.md
     └── Decomposing: references/patterns/decomposition.md

     │ Want to customize output?
     │
     ├── Focus:   references/templates/focus-output.md
     ├── Plan:    references/templates/plan-output.md
     ├── Spike:   references/templates/spike-output.md
     ├── Ship:    references/templates/ship-output.md
     ├── Review:  references/templates/review-output.md
     ├── Done:    references/templates/done-output.md
     ├── Drop:    references/templates/drop-output.md
     └── Status:  references/templates/status-output.md
```

**You don't need to read everything.** SKILL.md is the only file the agent loads. References are loaded on-demand when a phase triggers.

## How It Works

```
 YOU                          COMPACT-WORKFLOW                  OUTPUT
──────                        ───────────────                   ──────

 "compact focus"
      │
      ├───────────────────▶  Scan specs (active/backlog/dropped/shipped)
                             Ask goal (production/MVP/infra/custom)
                             Dispatch perspective agents
                             Rank findings ──────────────────▶  prioritized tasks
      │
 Select tasks
      │
      ├───────────────────▶  Create specs ───────────────────▶  specs/backlog/

 "compact spec {idea}"
      │
      ├───────────────────▶  Parse idea
                             User journey (MANDATORY)
                             Acceptance criteria (MANDATORY)
                             Scope ↔ AC mapping
                             Quality checklist
                             Plan review
                             Dev-readiness check ──────────▶  specs/active/{slug}.md
      │
 "compact ship"
      │
      ├───────────────────▶  Detect mode (ONE-SHOT / LOOP)
                             Resume state check
                             ┌─────────────────────┐
                             │     SHIP LOOP       │
                             │ build → review → fix│◄──┐
                             │ exit check (ACs)    │───┘ not clean
                             │ stuck detection     │
                             └────────┬────────────┘
                                      │ clean
                             Quality gates ────────────────▶  All ACs + scope [x]
      │
 "compact done"
      │
      ├───────────────────▶  Validate (ACs + tests + gates)
                             Capture retro
                             Propose memory update ────────▶  Agent config learning
                             Archive spec ─────────────────▶  specs/shipped/{slug}.md
      │
 YOU (manual)
      │
      ├─ git commit + push
      ├─ deploy
      └─ verify in production
```

### Lifecycle

```
┌──────────┐  compact   ┌──────────┐ compact  ┌──────────┐ review? ┌───────────┐
│ UNFOCUSED│──focus────▶│   IDEA   │──spec───▶│  DRAFT   │────────▶│ REVIEWING │
└──────────┘            └──────────┘          └──────────┘         └───────────┘
      ▲                                            ▲                    │
      │                                            │ revise [?][!]      │ ready?
      │                                            └────────────────────┘
      │                                                                 │
      │              ┌──────────┐                               ┌───────────┐
      │              │IMPLEMENTING│◄─────────── compact ship ───│ DEV_READY │
      │              └─────┬─────┘                              └───────────┘
      │                    │
      │              ┌───────────┐
      │              │   SHIP    │──── iterate
      │              │   LOOP    │
      │              └─────┬─────┘
      │                    │ clean          │ stuck
      │              ┌──────────┐    ┌──────────┐
      │  compact     │  READY   │    │ DROPPED  │
      └──────────────│  done    │    └──────────┘
                     └──────────┘     specs/dropped/
                          │
                     ┌──────────┐
                     │ SHIPPED  │
                     └──────────┘
                      specs/shipped/
```

### Quality Gates

```
 Quick Pass (per edit)                Full Pass (before exit)
┌──────────────────────┐     ┌─────────────────────────────────────────┐
│ LINT ──▶ TYPECHECK   │     │ LINT ──▶ TYPECHECK ──▶ BUILD ──▶ TEST  │
│(changed)  (changed)  │     │(changed)   (full)      (full)  (related)│
└──────────────────────┘     └─────────────────────────────────────────┘
```

### Review Notation

```
[?] unclear    [!] risk    [+] add to scope
[-] cut scope  [~] rephrase   [ok] approved
```

## Customization

Edit the skill files directly. Symlink at user-level or project-level, then modify to match your needs.

```
workflow/
├── SKILL.md ──────────────────── Task templates, review systems, risk thresholds
├── references/
│   ├── actions/                  ← Action logic (what the agent does)
│   │   ├── plan.md ───────────── Spec requirements, dev-readiness gates, review rounds
│   │   ├── ship.md ───────────── Quality gates, iteration limits, exit criteria
│   │   ├── review.md ─────────── Perspectives (add/remove/change levels)
│   │   ├── focus.md ──────────── Codebase scan, perspective weights, ranking formula
│   │   ├── done.md ───────────── Validation checklist, memory update settings
│   │   └── drop.md ───────────── Drop flow
│   │
│   ├── templates/                ← Output templates (what the agent returns to you)
│   │   ├── plan-output.md ────── Spec summary, codebase impact, analysis, readiness
│   │   ├── spike-output.md ───── Decision, findings, next step
│   │   ├── ship-output.md ────── Iteration progress, exit states, spec mutations
│   │   ├── review-output.md ──── Per-file findings, fix actions, verdict
│   │   ├── focus-output.md ───── Ranked tasks, perspective scores, readiness
│   │   ├── done-output.md ────── Validation results, retro, memory updates
│   │   ├── drop-output.md ────── Drop reason, learnings, reusable pieces
│   │   └── status-output.md ──── Current state, progress, suggested next action
│   │
│   ├── spec-template.md ─────── Spec structure, generation rules, validation
│   ├── quality-gates.md ──────── Gate levels (BLOCKING/ADVISORY/SKIP), commands
│   ├── session-management.md ── Progress tracking, resume, stuck detection
│   └── memory-update.md ─────── Memory update protocol, agent config targets
```

**Two layers of customization:**

| Layer | What to edit | Controls |
|-------|-------------|----------|
| **Actions** (`actions/*.md`) | Logic, gates, perspectives, limits | What the agent does |
| **Templates** (`templates/*-output.md`) | Output format, sections, wording | What the agent returns to you |

### Examples

**Change what the agent shows after plan:** Edit `templates/plan-output.md` — add/remove sections, change format.

**Change what review findings look like:** Edit `templates/review-output.md` — change format, add fields, adjust rules.

**Change quality gate commands:** Edit `quality-gates.md` — update auto-detection tables or add explicit commands.

**Add review perspectives:** Edit `actions/review.md` — add rows to the perspectives table.

**Skip typecheck:** Edit `quality-gates.md` — set Typecheck level to SKIP.

**Require manual spec review:** Edit `actions/plan.md` — set manual review gate to BLOCKING.

**Change iteration limits:** Edit `actions/ship.md` — update the "Default Iterations by Size" table.

**Change memory update behavior:** Edit `memory-update.md` — adjust categories, agent targets, or disable proposals.

---

## Quality Guard

compact-workflow gates every spec that modifies the skill itself with a **Quality Guard** (BLOCKING for dev-readiness and ship). Feature specs about user code are unaffected.

**What it catches:** silent regressions like lost source attribution, document boundary erosion (tone modes leaking into spec files / code / commits), narrowed security triggers, broken action router, quality gate bypass, broken loading mechanisms, AC ↔ scope traceability loss, irreversible coupling between improvements.

**Detection:** automatic. Spec touches `workflow/`, `CHANGES.md`, `NOTICE.md`, `LICENSE`, or `docs/` → skill-improvement → Quality Guard runs.

**Rules:** 10 BLOCKING (Tier 1) + 8 WARN-eligible (Tier 2) + 2 informational (Tier 3). Mini-tier specs apply Tier 1 only (10 rules); standard-tier specs apply Tier 1 + Tier 2 (18 rules). Pre-fill heuristic auto-marks trivially-N/A rules to reduce checklist fatigue.

**Disable:** no disable path. The Guard is BLOCKING by design — emergency-bypass is when quality slips most.

**Version:** pinned per spec (`quality-guard-version: v1` in frontmatter). Revisions audit against pinned version, not mutable HEAD.

Full rules, verdict rubric, violation report format, detection rule: see [`references/quality-guard.md`](references/quality-guard.md). Authoritative spec: `../specs/shipped/2026-04-30-skill-quality-guard.md`.

## Tone

compact-workflow outputs in **caveman tone** by default (level: `full`). Drops articles, filler, pleasantries, hedging. Fragments OK. Technical accuracy preserved. ~65-75% token reduction on agent responses.

**Boundaries (always normal prose):**
- Security warnings + destructive op confirmations (auto-clarity)
- Code blocks, commit messages, PR bodies
- Spec files, CHANGES.md, NOTICE.md, this README — anything written to disk

**Switch level:** `compact tone lite` (drop only filler) | `compact tone full` (default) | `compact tone ultra` (max compression)

**Disable:** Type `stop caveman` or `normal mode` in chat. Persists for session. Re-enable via `compact tone full`.

Rules sourced from [`JuliusBrussee/caveman`](https://github.com/JuliusBrussee/caveman) (MIT). Inline copy in `SKILL.md` `## Tone` section. Audit trail: `references/tone/caveman.md`.

## Attribution

Forked from `agent-skills/workflow` (upstream SHA `5f6f937ad7e0bd02d8cbfbf4e3db6e460f10e469`). MIT license preserved. See `NOTICE.md` and `../CHANGES.md` for divergences.

## Key Principles

- **Spec = source of truth**: Always updated, always reflects reality
- **ACs define done**: Work finishes when all Must Have + Error ACs pass
- **New code = new tests**: Every new behavior must have tests (BLOCKING)
- **Two review systems**: Plan review (specs) + dev review (code)
- **Quality gates**: Auto-detected, skippable — edit skill files to configure
- **Ship loop**: Build → review → fix with stuck detection + escalation
- **Risk-aware**: Agent flags HIGH risk changes for human review
- **Context-aware memory updates**: Done phase reads existing rules, proposes thinking patterns + coding rules + project rules
- **Human controls deployment**: Agent codes and validates, you push and deploy
- **16 patterns**: Implementation, planning, debugging, decisions, decomposition
- **Agent-portable**: Capability fallbacks for agents without task tracking or shell access
