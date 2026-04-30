# Caveman tone — source attribution

**Audit trail only. Not loaded at runtime.** Tone rules live inline in `workflow/SKILL.md` `## Tone` section.

## Source

- **Repo**: [`JuliusBrussee/caveman`](https://github.com/JuliusBrussee/caveman)
- **License**: MIT (compatible with compact-workflow MIT)
- **Source SHA copied from**: `84cc3c14fa1e10182adaced856e003406ccd250d` (HEAD as of 2026-04-30)
- **Source file**: `skills/caveman/SKILL.md`

## Why inline (not loaded as file)

Action router in `workflow/SKILL.md` only loads `references/actions/*.md` on phrase match. No load-trigger mechanism exists for tone overlays — a `## Tone` section pointing at this file would never be read by the agent during action execution. Rules must live inside `SKILL.md` to be loaded with the skill.

This file = audit trail only:
- Source attribution + SHA → drift detection
- Rationale → why we forked the rules into our SKILL.md
- License compliance → MIT requires attribution

## Drift policy

Caveman repo may update rules. Inline copy in `SKILL.md` does NOT auto-update. Manual sync required:

1. Pull caveman HEAD SHA: `git ls-remote https://github.com/JuliusBrussee/caveman.git HEAD`
2. Compare against SHA above
3. If different — read upstream `skills/caveman/SKILL.md`, diff against our inline copy
4. Decide: merge changes / hold position / revise locally
5. Update SHA above + `CHANGES.md` entry

No automatic sync. Drift acceptable if intentional.

## Rationale (why caveman tone for compact-workflow)

- compact-workflow output verbose by default (inherited from upstream agent-skills/workflow)
- Solo dev profile — fast read, terse signals over polish
- ~65-75% token reduction on agent responses (estimate from caveman repo benchmarks)
- Spec content + code + commits stay normal prose — readability of artifacts preserved
- Auto-clarity protects security warnings + destructive ops from compression-induced ambiguity

## Local divergences from upstream caveman

| Divergence | Why |
|------------|-----|
| Default level = `full` (not configurable per-action) | Solo dev, no per-action variance needed |
| Spec-file boundary added (caveman does not enforce) | compact-workflow writes spec files; caveman repo does not have this concept |
| Auto-clarity trigger list explicit + authoritative (caveman implicit) | Eliminates agent guessing — testable boundary for AC-2 |
| Removed: Wenyan mode (Chinese classical) | Not relevant for solo English-language workflow |
| Removed: `/caveman` slash command — replaced by `compact tone <level>` | Coexist with compact-workflow command convention |

See `workflow/SKILL.md` `## Tone` section for the live ruleset.
