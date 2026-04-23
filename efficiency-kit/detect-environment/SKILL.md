---
name: detect-environment
description: "One-pass environment detection for development tools. Use at the start of a coding session in a new project, or when a command fails with 'command not found'. Detects Python (system/venv/poetry/uv), Node.js, TypeScript, Dart/Flutter, Rust/Cargo, Git, Docker, and Java. Caches results to avoid repeated tool-location discovery chains."
---

# Detect Environment

Run the probe script at session start or when `command not found` occurs:

Resolve the helper from the installed skill directory, not from the current repo. Do not look for repo-local `scripts/env-probe.sh` unless the repo explicitly provides its own.

```bash
ENV_PROBE=""
for skill_dir in \
  "$HOME/.codex/skills/detect-environment" \
  "$HOME/.claude/skills/detect-environment" \
  "$HOME/.gemini/skills/detect-environment" \
  "$HOME/.config/opencode/skills/detect-environment"; do
  if [ -f "$skill_dir/scripts/env-probe.sh" ]; then
    ENV_PROBE="$skill_dir/scripts/env-probe.sh"
    break
  fi
done

if [ -n "$ENV_PROBE" ]; then
  bash "$ENV_PROBE"
else
  echo "detect-environment helper not installed; use command -v and version checks manually."
fi
```

## Output

JSON object with detected tools and their paths. Example:

```json
{
  "python": {"cmd": ".venv/bin/python", "version": "3.11.5"},
  "pip": {"cmd": ".venv/bin/pip", "version": "23.2"},
  "package_manager": "uv",
  "node": {"cmd": "/usr/bin/node", "version": "20.10.0"},
  "tsc": {"cmd": "npx tsc", "version": "5.3.2"},
  "git": {"cmd": "/usr/bin/git", "version": "2.43.0", "in_repo": true},
  "docker": {"cmd": "/usr/bin/docker", "version": "24.0.7"},
  "dart": null,
  "flutter": null,
  "cargo": null,
  "java": null
}
```

## Usage

- Use the `cmd` value from output to construct commands (e.g., use `.venv/bin/python` not `python`)
- `null` means the tool is not installed — don't try to use it
- Re-run after installing tools, switching project directories, or activating a virtual environment
- The script always exits 0 — missing tools are reported as null, not errors
- If the helper is not installed, perform equivalent manual `command -v` and version checks. Missing this optional helper is not itself a task failure.
