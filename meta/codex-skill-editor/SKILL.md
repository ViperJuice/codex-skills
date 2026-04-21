---
name: codex-skill-editor
description: "Codex skill editor. Use when the user wants to apply an improvement plan produced by codex-skill-improvement-planner to Codex skill files. Edits only targeted `codex-*` skills by default, archives consumed reflections after successful edits, and uses apply_patch for manual changes."
---

# Codex Skill Editor

Applies a structured improvement plan to Codex skill files. It is deliberately narrower than arbitrary skill editing: it consumes plans from `codex-skill-improvement-planner` and updates the named target skills.

## Core Rules

- Read the improvement plan and target `SKILL.md` before editing.
- Use `apply_patch` for manual edits.
- Edit only skills named by the plan.
- Default target set is `codex-*` skills. Do not edit the original Claude-oriented skills unless the plan explicitly names them and the user confirms that scope.
- Preserve skill frontmatter validity.
- Do not push or commit unless the user explicitly requests it.
- Archive consumed reflections only after all recommendations citing them succeeded.

## Inputs

- Plan path: explicit path, or latest `~/.codex/skills/codex-skill-improvement-planner/plans/plan-v*.md`.
- `--dry-run`: parse and report intended edits without changing files.

If no plan path is explicit, first check the current repo and branch handoff from `codex-skill-improvement-planner` using `runtime-state.md`: read `~/.codex/skills/codex-skill-improvement-planner/handoffs/<repo_hash>/<branch_slug>/latest.md`, validate `from`, `repo`, `repo_root`, `branch`, `branch_slug`, `commit`, and `artifact`, then use the artifact only if it exists under the current repo root. Ignore missing or mismatched handoffs unless the user explicitly asks to reuse cross-branch state.

## Workflow

1. Resolve and read the plan.
2. Parse:
   - `reflections_consumed`;
   - recommendations by skill;
   - cross-cutting recommendations;
   - contradictions.
3. If contradictions exist, stop and ask the user how to resolve them unless the plan already contains a resolution.
4. Validate target skills:
   - source path under `planning-chain and meta/<skill>/SKILL.md` when working in this dotfiles repo;
   - symlink/runtime path under `~/.codex/skills/<skill>/SKILL.md` only when no source path exists.
5. For `--dry-run`, report the target files and recommendation summaries, then stop.
6. Apply recommendations:
   - group changes per target skill to avoid conflicting edits;
   - keep `SKILL.md` concise;
   - move lengthy examples into `references/`;
   - update `agents/openai.yaml` when display metadata becomes stale.
7. Validate:
   - YAML frontmatter parses;
   - `name` matches the skill directory intent;
   - `description` clearly states trigger scope and non-scope;
   - referenced files exist.
8. Archive reflections:
   - move successfully consumed reflection files to an `archive/` directory under the same repo and branch subtree;
   - leave reflections in place for failed recommendations.

## Failure Policy

- Malformed plan: stop and report exact parse failure.
- Missing target skill: mark that recommendation failed; continue only if other independent targets remain.
- Patch conflict: re-read the file, adjust once, then report if still blocked.
- Validation failure: fix if local to the edit; otherwise roll forward with a clear report and do not archive affected reflections.

## Closeout

Report:

- applied recommendations;
- skipped or failed recommendations;
- files changed;
- reflections archived;
- validation commands run.

If writing self-improvement state, follow `runtime-state.md` and use Codex paths only:

- Reflection: `~/.codex/skills/codex-skill-editor/reflections/<repo_hash>/<branch_slug>/<run_id>.md`
- Handoff: `~/.codex/skills/codex-skill-editor/handoffs/<repo_hash>/<branch_slug>/<run_id>.md`
- Latest handoff pointer: `~/.codex/skills/codex-skill-editor/handoffs/<repo_hash>/<branch_slug>/latest.md`

Handoff frontmatter must include `from: codex-skill-editor`, `timestamp:`, `repo:`, `repo_root:`, `branch:`, `branch_slug:`, `commit:`, `run_id:`, and `artifact:`. Update `latest.md` with the same handoff content.
