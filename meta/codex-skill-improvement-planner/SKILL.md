---
name: codex-skill-improvement-planner
description: "Codex skill feedback aggregator. Use when the user wants to review Codex skill reflections, aggregate recurring feedback, or plan improvements to `codex-*` skills. Produces an improvement plan for codex-skill-editor and does not edit skills itself."
---

# Codex Skill Improvement Planner

Aggregates reflection files for Codex skills and produces a structured improvement plan. It does not edit skills; `codex-skill-editor` applies the plan.

## Core Rules

- Planning only. Do not modify `SKILL.md` files.
- Prefer Codex skill state under `~/.codex/skills/<skill>/reflections/**`.
- Also inspect source-controlled reflections under `planning-chain and meta/codex-*/reflections/**` when present.
- Exclude any path with an `archive/` component.
- Follow the recursive reflection rules in `runtime-state.md`.
- Act only on recurring evidence unless the user explicitly asks to apply one-off feedback.
- Do not spawn subagents unless the user explicitly asks for delegated analysis.

## Inputs

- `--target <skill-name>`: plan for one skill.
- `--min-reflections <N>`: default `2`.
- `--output <path>`: default `~/.codex/skills/codex-skill-improvement-planner/plans/plan-v<N>-<ISO>.md`.

## Workflow

1. Enumerate unarchived reflections for:
   - `codex-phase-roadmap-builder`
   - `codex-plan-phase`
   - `codex-execute-phase`
   - `codex-task-contextualizer`
   - `codex-skill-improvement-planner`
   - `codex-skill-editor`
   - `codex-plan-detailed`
2. Parse each reflection:
   - skill name;
   - version or timestamp;
   - `What worked`;
   - `Improvements to SKILL.md`;
   - raw body when headings are missing.
3. Gate on minimum evidence:
   - zero reflections: report that there is nothing to aggregate;
   - fewer than `--min-reflections` for a skill: record as skipped.
4. Aggregate recommendations:
   - group recurring themes by skill;
   - separate actionable changes from speculative notes;
   - flag contradictions instead of resolving them silently;
   - reject repo-specific recommendations unless the target skill is intentionally repo-specific.
5. Produce a plan with:
   - frontmatter listing consumed reflection paths;
   - recommendations by skill;
   - cross-cutting recommendations;
   - speculative notes;
   - contradictions;
   - archival directive for `codex-skill-editor`.

## Plan Format

```markdown
---
from: codex-skill-improvement-planner
timestamp: <ISO>
min_reflections: <N>
reflections_consumed:
  - <absolute path>
---

# Codex skill improvement plan — <ISO>

## Summary

## Recommendations by skill

### <skill-name>
- **Change**: <directive>
  - **Rationale**: <evidence>
  - **Supporting reflections**: <ids>

## Cross-cutting recommendations

## Speculative / low-confidence notes

## Contradictions surfaced

## Archival directive for codex-skill-editor
```

## Closeout

In Default mode, write the plan only if the user asked for an artifact. Otherwise summarize the recommendations. Do not archive reflections; that is the editor's job.

If writing self-improvement state, follow `runtime-state.md` and use Codex paths only:

- Reflection: `~/.codex/skills/codex-skill-improvement-planner/reflections/<repo_hash>/<branch_slug>/<run_id>.md`
- Handoff: `~/.codex/skills/codex-skill-improvement-planner/handoffs/<repo_hash>/<branch_slug>/<run_id>.md`
- Latest handoff pointer: `~/.codex/skills/codex-skill-improvement-planner/handoffs/<repo_hash>/<branch_slug>/latest.md`

Handoff frontmatter must include `from: codex-skill-improvement-planner`, `timestamp:`, `repo:`, `repo_root:`, `branch:`, `branch_slug:`, `commit:`, `run_id:`, and `artifact:`. Update `latest.md` with the same handoff content.
