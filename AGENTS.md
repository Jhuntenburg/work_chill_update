# Repository Guidelines

## Project Structure & Module Organization
- `work_chill`: primary Bash CLI and background worker logic (start/pause/resume/status/shutdown flows).
- `chill_render_markdown`: helper script that renders `work_chill_usage.md` to HTML for quick local viewing.
- `work_chill_usage.md`: end-user command guide.
- `storyDocs/work_chill_handoff_status.md`: deeper behavioral and logging notes.
- Runtime artifacts are not stored in-repo; they live under `~/.work_chill/` (`state.sh`, `bg.log`, `logs/YYYY-Www.jsonl`).

## Build, Test, and Development Commands
- `install -m 755 ./work_chill "$HOME/bin/work_chill"`: install/update the CLI in `~/bin`.
- `work_chill doctor`: environment and runtime health checks (`doctor_result: healthy` expected).
- `bash -n ./work_chill`: syntax validation for the main script.
- `WORK_CHILL_TEST=1 work_chill start`: run a short-timer test session (10s/120s timings).
- `work_chill status` and `work_chill tail`: inspect process state and recent logs while validating changes.

## Coding Style & Naming Conventions
- Bash-first project; keep changes compatible with macOS shell tools (`osascript`, `date -j`, `open`).
- Match existing style: 2-space indentation, `lower_snake_case` for functions/locals, `UPPER_SNAKE_CASE` for constants.
- Keep command surface stable (`start`, `pause`, `resume`, `debug-prompt`, etc.) and update help/docs when adding commands.
- Preserve JSONL event names and keys in logs to avoid breaking status/welcome-back parsing.

## Testing Guidelines
- No formal test framework is committed; use repeatable smoke tests.
- Minimum validation after behavior changes:
  1. `bash -n ./work_chill`
  2. `work_chill doctor`
  3. `WORK_CHILL_TEST=1 work_chill start`, then exercise `pause`, `resume`, `status`, `shutdown`.
- Confirm logs are written to `~/.work_chill/logs/YYYY-Www.jsonl` and `~/.work_chill/log.jsonl` points to the current week.

## Commit & Pull Request Guidelines
- Follow existing commit tone: short, imperative summaries; optional scope prefixes are common (example: `docs(work_chill): document weekly log files and symlink`).
- Keep commits focused (script logic, docs, or tooling) and include doc updates with behavior changes.
- PRs should include: what changed, why, commands run for validation, and sample output when relevant (`doctor`, `status`, log excerpts).
