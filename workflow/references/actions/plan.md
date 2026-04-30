# Plan Action

> **Agent:** Load this file when `plan` triggers. Also load `references/spec-template.md` — it defines spec structure and validation rules. Also load `references/rules-discovery.md` — plans must respect project/user conventions. Also load `references/quality-guard.md` — gates skill-improvement specs (BLOCKING for dev-readiness when applicable).

**BLOCKING: Never delegate planning to the host agent's built-in plan mode.**
This skill manages its own planning by writing spec files to `specs/active/`.
Always stay in the agent's normal execution mode — read, write, and execute directly.

Parse idea, create spec, estimate effort. Output: spec file ready for `ship`.

**The spec template (`references/spec-template.md`) is the SINGLE SOURCE OF TRUTH for spec structure, generation rules, and validation.** This file defines the orchestration — template defines the content.

---

## Task Tracking (MANDATORY for mini/standard)

Before starting planning, create tasks in your agent's built-in task/todo system:

```
[ ] Parse idea and identify scope
[ ] Discover test environment (read existing tests, identify patterns)
[ ] Codebase impact analysis (read affected code, map dependencies)
[ ] Generate spec (following spec-template.md generation order)
[ ] Generate test strategy (AC mapping + failure mode tests + mock avoidance plan)
[ ] Spec analysis (adversarial review of generated spec)
[ ] Run dev-readiness check (spec-template.md validation rules)
[ ] All dev-readiness gates PASS
```

Update tasks as you work. One in-progress at a time.

## Step 0: Detect Project + Test Environment

```
Check project for:
  - Monorepo (turbo.json)
  - Package manager (pnpm/yarn/bun/npm from lockfile)
  - Test runner (vitest/jest/pytest from config)
  - E2E framework (playwright/cypress/maestro/detox from config)
  - Framework (next/expo/convex from package.json)

Deep test discovery (feeds spec Test Strategy):
  - Read 3+ existing test files → understand patterns, conventions, helpers
  - Identify: test DB strategy, mock inventory, existing E2E tests, test utilities
  - Produce "Test Environment Profile" for spec's Test Strategy section
  - If no test infra: flag "not configured" with specific recommendation
  - If no E2E infra but unit tests exist: flag "partial — E2E setup needed at ship"
```

### TypeScript Structural Analysis (Optional)

If TypeScript detected (`tsconfig.json` exists):

```
Load references/codebase-intelligence.md for detection gate and command reference.
Run: npx codebase-intelligence overview --json <project-root>
  → Captures: file count, module count, dependency graph shape, top-level metrics
  → Store as "CI Snapshot" for Codebase Impact section
If CI unavailable: skip — grep/glob/read discovery continues as normal
```

## Step 1: Parse Idea

Extract from user input:
- **What**: Feature/fix description
- **Why**: Problem being solved
- **Who**: User persona affected
- **Scope signal**: Size estimate (trivial/micro/mini/standard)

## Step 2: Select Thinking Pattern

| Situation | Pattern |
|-----------|---------|
| Clear requirements | Chain-of-thought — step by step |
| Multiple approaches | Tree-of-thought — enumerate, score, choose |
| After failure | Reflexion — record failure, adjust, retry |
| Large scope | Decomposition — split into independent parts |

## Step 3: Generate Spec

**Follow `references/spec-template.md` exactly.** The template defines:
- Generation order (context → codebase → journey → ACs → scope → checklist → test strategy → risks → state → analysis)
- Per-section generation rules
- Traceability requirements
- Validation rules

Key principle: **Read the codebase BEFORE writing the spec.** Codebase impact analysis is MANDATORY and informs every section that follows.

**TypeScript projects with CI available** (augments grep/glob/read, does not replace):

```
For each file identified in Codebase Impact table:
  npx codebase-intelligence file --json <file>
    → Adds: imports, exports, coupling score, complexity
  npx codebase-intelligence dependents --json <file>
    → Adds: files affected by changes → populates AFFECTED rows
For key symbols being changed:
  npx codebase-intelligence impact --json <symbol>
    → Adds: blast radius data for Breaking Changes assessment
If CI unavailable: use grep for imports/exports, manual trace for dependents
```

Create spec file at `specs/active/{YYYY-MM-DD}-{slug}.md`.

### Tier Selection

| Tier | Size | Spec |
|------|------|------|
| trivial | <5 LOC | None — just implement |
| micro | <30 LOC | Inline comment in code |
| mini | <100 LOC | Mini template (see spec-template.md) |
| standard | 100+ LOC | Full template (see spec-template.md) |

### Pre-Mortem (Standard Tier)

Assume the feature failed. Why?

```
For each risk:
  Impact: HIGH/MEDIUM/LOW
  Likelihood: HIGH/MEDIUM/LOW
  Mitigation: What prevents this?
  Kill criteria: When do we abandon this approach?
```

For stateful features, include state machine analysis (see spec-template.md).

## Step 3.3: Generate Test Strategy (MANDATORY for mini/standard)

After spec generation, before analysis. Write the Test Strategy section into the spec.

```
1. Include Test Environment Profile from Step 0 discovery
2. Map each AC to test type:
   - Pure function (no I/O, no side effects) → unit test
   - Everything else → E2E test (default)
3. Map each Error Journey (E1, E2...) → E2E test intention
4. Map each Edge Case (EC1, EC2...) → E2E test intention
5. Mock avoidance plan — for each external dependency:
   - Does it have a sandbox/test mode? → use real
   - Docker image available? → containerize
   - In-memory equivalent? → use it
   - None available? → mock with justification
6. Write Test Strategy section into spec (see spec-template.md)
```

Failure hypotheses from Step 3.5 will feed back into Test Strategy (see below).

## Step 3.5: Spec Analysis (MANDATORY for mini/standard)

After generating the spec and Test Strategy, run adversarial analysis to populate the Analysis section.

**Failure hypothesis → Test Strategy feedback:** After analysis generates failure hypotheses, add defensive E2E test for each HIGH/MED hypothesis to the Test Strategy section. Update the Failure Mode Tests table.

### Mode by Tier

| Tier | Mode | Agents | Min Requirements |
|------|------|--------|------------------|
| mini | Inline | None (single-agent) | 3 assumptions, 1-2 blind spots, 1 failure hypothesis |
| standard | Parallel sub-agents | 2-3 | 3-5 assumptions, 2-4 blind spots, 3 failure hypotheses |

### Mini Mode (inline)

Apply 4 techniques sequentially (single agent, no sub-agents):

1. **Challenge assumptions**: List 3+ assumptions from the spec. For each: evidence for, against, verdict (VALID/RISKY/WRONG)
2. **Find blind spots**: What hasn't been considered? 1-2 items with category + impact
3. **Failure hypothesis**: IF {trigger} THEN {failure} BECAUSE {cause}. At least 1.
4. **Reframe**: Is this solving the right problem? Confirm or reframe.

Write results directly into spec's `## Analysis` section.

### Standard Mode (parallel sub-agents)

1. Read the spec's context, scope, codebase impact, and risks
2. Self-select 2-3 analysis perspectives most relevant to THIS spec's domain:
   - Always include 1 **Skeptic** (domain-agnostic adversarial thinker)
   - Add 1-2 domain-relevant perspectives (e.g., Security Engineer for auth, UX Designer for UI, Data Engineer for migrations)
3. Launch all perspectives as **parallel sub-agents** (single message, multiple Task tool calls)
4. Each agent receives: full spec + relevant codebase files from Codebase Impact
5. Each agent produces: assumptions table, blind spots, failure hypotheses, reframe
6. Synthesize: merge findings, deduplicate, rank by severity
7. Write synthesized results into spec's `## Analysis` section

### Rules

- Cross-reference against actual codebase (not just spec text)
- Every finding must be action-driven: `→ update spec` / `→ explore` / `→ question` / `→ no action`
- No passive observations — each finding forces a decision
- If analysis reveals spec gaps → update spec before proceeding to Step 4

## Step 3.7: Quality Guard Detection (CONDITIONAL — skill-improvement specs only)

**Load `references/quality-guard.md` if not already loaded.**

After Analysis is written, run the Detection Rule from `quality-guard.md`:

```
A spec is skill-improvement if its Codebase Impact table contains ANY of:
  - Path under `workflow/`
  - Repo-root file: `CHANGES.md`, `NOTICE.md`, `LICENSE`
  - Path under `docs/`

Hybrid spec (touches both skill paths AND user code) → classified as skill-improvement.
```

If detection does NOT match → skip this step entirely (zero overhead for feature specs).

If detection MATCHES:

1. Add `quality-guard-version: v1.1` (current HEAD) to spec frontmatter
2. Append `## Quality Guard Results` section per template in `references/spec-template.md`
3. Apply tier-conditional enforcement:
   - mini / trivial / micro → Tier 1 only (10 BLOCKING rules)
   - standard → Tier 1 + Tier 2 (10 BLOCKING + 8 WARN-eligible rules)
   - Tier 3 always informational
4. Run pre-fill heuristic from `quality-guard.md`:
   - QG-1 → N/A if no external source referenced
   - QG-4 → N/A if no router file edits
   - QG-7 → N/A if no new file references in SKILL.md
   - QG-12 → N/A if no stateful overlay introduced
   - QG-17 → N/A if no new triggers added
5. Mark all other required-tier rules as `[ ] PENDING` for user/agent to fill
6. Auditor narrates verdicts using `[QG-VIOLATION]` prefix marker for any FAIL (auto-clarity → normal prose)

**Dev-readiness blocks until `## Quality Guard Results` is GUARD-CLEAN:**
- Every Tier 1 rule = `PASS` or `NOT_APPLICABLE`
- Every Tier 2 rule (standard tier only) = `PASS`, `NOT_APPLICABLE`, or `WARN` with concrete justification
- No `FAIL`, no `PENDING`

## Step 4: Spec Review

Before finalizing, self-verify:

| Check | Question |
|-------|----------|
| Clarity | Could another developer implement from this spec alone? |
| Scope | Can this ship today? Anything cuttable? |
| Testability | Is every scope item verifiable? Does every failure mode have a planned test? |
| Test Strategy | Does Test Strategy exist? Are all ACs mapped? Are failure hypotheses covered? |
| Risks | Are kill criteria defined for high risks? |
| Completeness | Quality checklist covers edge cases? |
| Codebase | Did I actually read the code? Are affected files mapped? |

If any check fails, revise the spec before outputting.

## Step 5: Manual Review Gate

**BLOCKING for standard tier.** Pause for human approval before dev-readiness check.

When user requests review ("review the spec", "let me check") or tier is standard, pause for human review before dev.

### Review Notation

Reviewers annotate the spec with inline markers:

| Marker | Meaning |
|--------|---------|
| `[?]` | Unclear — needs clarification |
| `[!]` | Risk — not addressed or underestimated |
| `[+]` | Missing — should be added to scope |
| `[-]` | Cut — remove from scope (over-engineered) |
| `[~]` | Rephrase — intent is right, wording is wrong |
| `[ok]` | Approved — explicitly signed off |

### Review Flow

```
Spec created
     │
     ▼
Present spec for review
     │
     ├─ All [ok], no [?]/[!] ──▶ READY
     │
     ├─ Has [?] or [~] ────────▶ NEEDS_WORK → revise, re-present
     │
     ├─ Has [!] ───────────────▶ NOT_READY → address risks, re-review
     │
     └─ Has [+] or [-] ────────▶ CONDITIONAL → apply scope changes, re-score
```

Max 3 rounds. After round 3: must reach READY or user says "proceed anyway".

## Step 6: Dev-Readiness Check

**BLOCKING gate before `ship` can start.**

Run the validation rules from `references/spec-template.md` (Validation Rules table). All 14 rules must pass. Rule 15 applies to skill-improvement specs only — if applicable, GUARD-CLEAN required.

### Readiness Algorithm

```
ready_to_dev(spec):
  Run all validation rules from spec-template.md

  mandatory_fails = rules 1-8 that fail
  standard_fails = rules 9-14 that fail
  guard_fails = rule 15 (skill-improvement specs only) — Quality Guard NOT GUARD-CLEAN

  IF len(all_fails) == 0:
    RETURN READY
  ELIF mandatory_fails OR guard_fails:
    RETURN NOT_READY (cannot proceed)
  ELIF only standard_fails:
    RETURN CONDITIONAL (can proceed with acknowledgment)
```

**Rule 15 specifics:** If spec is classified as skill-improvement (per Detection Rule in `references/quality-guard.md`), the `## Quality Guard Results` section MUST be GUARD-CLEAN. Any Tier 1 FAIL or PENDING blocks dev-readiness. Tier 2 FAIL also blocks (Tier 2 may WARN with justification, but cannot FAIL).

Additional checks (beyond template validation):

```
  # Estimate within same-day
  IF estimate > 8h:
    WARN "Exceeds same-day. Split or cut scope."

  # Risks addressed (standard tier)
  IF tier == standard AND unmitigated HIGH risks:
    BLOCK "Unmitigated HIGH risks: {list}"

  # Dependencies resolved
  IF external blockers (APIs, access, data):
    BLOCK "Unresolved dependencies: {list}"

  # Manual review (if requested)
  IF review_required AND verdict != READY:
    BLOCK "Spec review not approved"
```

## Step 7: Output

**Follow the output template in `references/templates/plan-output.md`.**

Always return: file path, status, codebase impact, analysis (improvements + gaps + questions + risks), readiness scorecard.

### Dev-Readiness State Diagram

```
                   plan {idea}
                       │
                       ▼
                 ┌───────────┐
                 │   DRAFT   │
                 │ (spec     │
                 │  created) │
                 └─────┬─────┘
                       │
          ┌────────────┼────────────┐
          │            │            │
     (user asks    (auto-check)  (trivial/micro)
      for review)      │            │
          │            │            │
          ▼            ▼            ▼
   ┌────────────┐ ┌─────────┐  ┌───────┐
   │  REVIEWING │ │ CHECKING│  │ READY │──▶ ship
   │  (human)   │ │ (auto)  │  │ (skip)│
   └──────┬─────┘ └────┬────┘  └───────┘
          │             │
     [?][!][-][+]  blockers?
          │             │
     ┌────┴────┐   ┌───┴────┐
     │ Revise  │   │  FIX   │
     │ (max 3) │   │blockers│
     └────┬────┘   └───┬────┘
          │             │
          ▼             ▼
   ┌────────────┐ ┌─────────┐
   │  APPROVED  │ │  CLEAR  │
   │  (verdict: │ │ (0      │
   │   READY)   │ │ blockers│
   └──────┬─────┘ └────┬────┘
          │             │
          └──────┬──────┘
                 ▼
           ┌───────────┐
           │ DEV_READY │──▶ ship
           └───────────┘
```

## Tier Bypass

- **trivial** (<5 LOC): Skip spec creation entirely. Just implement.
- **micro** (<30 LOC): Inline spec as code comment. No file.
- **mini** (<100 LOC): Mini template from spec-template.md.
- **standard** (100+ LOC): Full template from spec-template.md.
- **Emergency**: User says "emergency fix" or "hotfix" → skip spec, log reason in history.
