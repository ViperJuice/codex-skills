---
name: validate-before-bash
description: "Pre-validation checks before running build tools and type checkers. Use before running tsc, dart analyze, flutter build, pytest, python -m compile, or similar build/check commands. Runs prerequisite checks (tool installed, config file exists, dependencies resolved) to prevent predictable failures. Also use when starting work in a new project to verify the build environment."
---

# Validate Before Bash

## When to Run Preflight

Run the preflight script in these situations:
- Before the FIRST `tsc`, `dart analyze`, `flutter build`, `pytest`, `cargo build` in a session
- After switching to a different project directory
- After `git checkout` to a different branch (dependencies may differ)
- When a command fails with exit 127 (command not found) or missing config errors

NOT needed for: `ls`, `git status`, `cat`, `grep`, or other basic commands.

## Quick Start

Resolve the helper from the installed skill directory, not from the current repo. Do not report a missing repo-local `scripts/preflight.sh` as a validation failure.

```bash
MODE="${1:-auto}"  # auto, typescript, python, dart, or rust
PREFLIGHT=""
for skill_dir in \
  "$HOME/.codex/skills/validate-before-bash" \
  "$HOME/.claude/skills/validate-before-bash" \
  "$HOME/.gemini/skills/validate-before-bash" \
  "$HOME/.config/opencode/skills/validate-before-bash"; do
  if [ -f "$skill_dir/scripts/preflight.sh" ]; then
    PREFLIGHT="$skill_dir/scripts/preflight.sh"
    break
  fi
done

if [ -n "$PREFLIGHT" ]; then
  bash "$PREFLIGHT" "$MODE"
else
  echo "validate-before-bash helper not installed; perform manual tool/config/dependency checks."
fi
```

- `auto` mode (default): detects language from config files in the current directory
- Output is JSON: `{"ready": true/false, "language": "...", "issues": [...]}`
- Exit 0 if ready, exit 1 if issues found
- If the helper is not installed, do the manual checks below and list the helper under "commands not run and why"; this alone is not a build or implementation failure.

## What It Checks

For each language:
- **Tool exists** on PATH (e.g., `npx`, `python3`, `cargo`)
- **Config file present** (tsconfig.json, pyproject.toml, pubspec.yaml, Cargo.toml)
- **Dependencies installed** (node_modules, .venv, .dart_tool, target/)
- **Basic version output** works (tool isn't broken)

## Using Results

If the script reports issues:
1. Fix each issue before running the build command
2. Common fixes: `npm install`, `pip install -e .`, `dart pub get`, `cargo fetch`
3. If the tool isn't installed, tell the user — don't try to install system tools without asking

If the script itself cannot be found, manually check the relevant command, config file, and dependency directory for the project language. When the requested build/test verification passes, do not mark the task failed only because this optional helper was unavailable.

## Reference

See `references/environment-checklist.md` for detailed per-language checklists covering version requirements and build-specific flags.
