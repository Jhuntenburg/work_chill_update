# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

`work_chill` is a single-file Bash script (~900 lines) that acts as a personal macOS work-pacing helper. It runs friction-check prompts via AppleScript dialogs every 55 minutes, tracks active work time with pause/resume, triggers escalation flows when stuck, and runs a guided shutdown ritual at end-of-day.

## Key Files

- `work_chill` — the main Bash script (installed to `~/bin/work_chill`)
- `chill_render_markdown` — renders `work_chill_usage.md` to HTML and opens in browser
- `work_chill_usage.md` — end-user usage guide
- `storyDocs/work_chill_handoff_status.md` — detailed handoff doc with full behavior spec, state layout, logging schema, and validation history

## Install & Validate

```bash
install -m 755 ./work_chill "$HOME/bin/work_chill"
work_chill doctor          # expect: doctor_result: healthy
bash -n ./work_chill       # syntax check without running
```

## Test Mode

```bash
WORK_CHILL_TEST=1 work_chill start
```

Shortened timers: 10s prompt interval, 120s day length, 15s meditation, 10s escalation break. Use `WORK_CHILL_FORCE_ACTION=<action>` with `debug-prompt` to simulate specific friction-check responses (e.g. `same_as_last`, `stuck`, `new_task`).

## Runtime State & Logs

All runtime data lives under `~/.work_chill/`:
- `state.sh` — sourced shell variables (day_start_ts, prompt_pid, eod_pid, paused, pause timers, consecutive_same_count, etc.)
- `logs/YYYY-Www.jsonl` — ISO-week event logs (JSONL, one event per line)
- `log.jsonl` — symlink to current week file (compatibility)
- `bg.log` — background worker diagnostics

## Architecture Notes

- The script uses `set -u` (not `set -e`) — undefined variable access fails but command errors don't abort.
- Background workers (prompt loop + EOD timer) are spawned via `nohup` + `disown` and tracked by PID in `state.sh`.
- `start` is restart-safe: kills existing workers before launching new ones.
- `pause` kills workers and records pause timestamp; `resume` computes delta, adds to `paused_total_secs`, and reschedules workers from remaining active time.
- AppleScript `display dialog` is limited to 3 buttons by macOS; timeout/cancel maps to "Skip" or "Ignore" as appropriate.
- On AppleScript failure, the script falls back to terminal prompts and sends a macOS notification.
- Welcome-back on `start` scans SHUTDOWN events in reverse across weekly logs (up to 8 weeks back, then legacy fallback). Uses `jq` if available, otherwise a pure-shell JSON parser.

## Logging Schema (event-specific keys)

Each JSONL line has `ts`, `event`, `action`. Additional keys vary:
- `TASK_PULSE` adds `task_text`; `SHUTDOWN` adds `task_text`, `tomorrow_first_step`, `open_items`
- `PAUSE` adds `active_elapsed_secs`; `RESUME` adds `paused_delta_secs`
- `ESCALATION` adds `choice` and optionally `blocker_text` or `next_step_text`

## Development Guidelines

- After any change, re-install with `install -m 755` and re-run `work_chill doctor` + `work_chill status`.
- Preserve the JSONL log format and `state.sh` variable names — the welcome-back parser and status command depend on them.
- Keep the script self-contained (single file, no external dependencies beyond standard macOS tools + optional `jq`).
- `--end-at HH:MM` uses local time; if the time has already passed today, it schedules for the next day.
