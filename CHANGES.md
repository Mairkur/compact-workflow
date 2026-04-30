# Divergences from upstream agent-skills/workflow

Each entry: file, type (NEW | EDIT-frontmatter | EDIT-body | APPEND | COPY), reason.
Future merger reads this to resolve conflicts with correct intent.

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
- <append entries here per improvement spec>
