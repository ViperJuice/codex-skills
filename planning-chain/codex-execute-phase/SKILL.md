---
name: codex-execute-phase
description: "Codex-optimized executor for a `codex-plan-phase` lane plan. Use when the user wants Codex to implement a planned phase. Executes lanes with clean git preflight, owned-file boundaries, verification, and optional explicit worker-subagent fanout for disjoint lanes."
---

# Codex Execute Phase

Executes a phase plan produced by `codex-plan-phase`. The default executor is the main Codex thread. Worker subagents are optional and only used when the user explicitly asks for subagents, delegation, or parallel execution.

## Core Rules

- Read the full phase plan before editing.
- Preserve user work. Never revert changes you did not make.
- Use `apply_patch` for manual edits.
- When implementation depends on current external documentation, use PMCP first. Context7 is the preferred path for library docs; Bright Data or other search/scrape tools may be used only when `gateway_catalog_search` shows they are available.
- Keep lane ownership boundaries. If implementation requires touching another lane's files, stop and revise the plan or ask the user.
- Do not run destructive git commands such as `git reset --hard` or `git checkout -- <path>` unless the user explicitly requested that operation.
- Do not commit, push, or merge unless the user asked for those git actions.

## Inputs

- Plan path: default latest `plans/phase-plan-*.md`.
- `--dry-run`: parse and print the lane schedule without editing.
- `--parallel`: allowed only when the user explicitly requests parallel worker execution.

If no plan path is explicit, first check the current repo and branch handoff from `codex-plan-phase` using `codex-config/shared/runtime-state.md`: read `~/.codex/skills/codex-plan-phase/handoffs/<repo_hash>/<branch_slug>/latest.md`, validate `from`, `repo`, `repo_root`, `branch`, `branch_slug`, `commit`, and `artifact`, then use the artifact only if it exists under the current repo root. Ignore missing or mismatched handoffs unless the user explicitly asks to reuse cross-branch state.

## Preflight

1. Resolve repo root and plan path.
2. Run `git status --short`.
3. Run `git status --short -- <plan_path>` and classify the plan as tracked, modified, or untracked. If untracked, warn before editing or executing: `This execution input is untracked and can be deleted by git clean -fd.`
4. If the tree has unrelated dirty files, leave them alone and scope edits around them.
5. Parse:
   - interface gates;
   - lane DAG;
   - owned files;
   - task lists;
   - verification commands.
6. Validate producer dependencies:
   - any lane that consumes another lane's findings, interfaces, or artifacts must list that producer lane in `Depends on`;
   - any lane that writes a synthesized artifact must be downstream of every producer lane it summarizes;
   - if dependencies are missing, stop and require a plan correction before execution.
7. For `--dry-run`, report the topological lane order and stop.

## Execution Workflow

1. Execute lanes in topological order.
2. For each lane:
   - read the owned files and related tests;
   - write or update tests first when practical;
   - implement only lane-scoped changes;
   - run lane verification;
   - run any phase-level checks that cover touched files.
3. After each lane:
   - inspect `git diff -- <owned files>`;
   - confirm no peer-owned files were modified;
   - record completed gates.
4. After all lanes:
   - run the full phase verification commands;
   - summarize changed files, tests run, and any residual risks.

## Optional Worker Fanout

Use worker subagents only when the user explicitly authorizes parallel agent work and the DAG-ready lanes are disjoint.

For every worker brief:

- State that they are not alone in the codebase.
- Assign exact owned files or globs.
- List files they may read but not edit.
- Tell them not to revert unrelated changes.
- Require a final response with changed paths, tests run, failures, and blockers.
- Do not give two workers overlapping write ownership.

The main thread remains responsible for integrating results, reviewing diffs, running final verification, and resolving conflicts.

## Failure Policy

- Test failure in a lane: diagnose once, fix within the lane if the cause is local, otherwise stop and report.
- Ownership violation: stop and revise the plan before continuing.
- Missing dependency or unclear interface: stop and ask for clarification or update the plan.
- Verification command unavailable: report the missing tool and run the closest available static check only if it is meaningful.

## Closeout

If execution updates the plan or roadmap, run `git status --short -- <plan_path> <roadmap_path>` before final closeout. Report the tracking state for every consumed or updated planning artifact. If any planning artifact is untracked, ask whether to stage it with `git add` or record an explicit `untracked artifact risk` in the handoff. Do not describe execution as safely closed out while the only execution record is untracked.

Report:

- lanes completed;
- files changed;
- planning artifact tracking state;
- verification commands and results;
- commands not run and why;
- follow-up risks or manual checks.

If writing self-improvement state, follow `codex-config/shared/runtime-state.md` and use Codex paths only:

- Reflection: `~/.codex/skills/codex-execute-phase/reflections/<repo_hash>/<branch_slug>/<run_id>.md`
- Handoff: `~/.codex/skills/codex-execute-phase/handoffs/<repo_hash>/<branch_slug>/<run_id>.md`
- Latest handoff pointer: `~/.codex/skills/codex-execute-phase/handoffs/<repo_hash>/<branch_slug>/latest.md`

Handoff frontmatter must include `from: codex-execute-phase`, `timestamp:`, `repo:`, `repo_root:`, `branch:`, `branch_slug:`, `commit:`, `run_id:`, and `artifact:`. Put open follow-up items in the body, and update `latest.md` with the same handoff content.
