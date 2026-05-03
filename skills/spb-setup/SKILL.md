---
name: spb-setup
description: Sets up the SynthPanel BMad module in a project. Use when the user requests to 'install synthpanel', 'configure SynthPanel', 'setup spb module', or 'install panel facilitator'.
---

# SynthPanel Module Setup

## Overview

Installs and configures the SynthPanel BMad expansion pack into a project. Module identity (`code: spb`, version, schema range) comes from `./assets/module.yaml`. Collects user preferences and writes them to:

- **`{project-root}/_bmad/config.yaml`** — shared project config: core settings at root plus a `spb:` section with module metadata and module-specific values (`verdict_output_path`, `readback_card_path`, `mcp_server_url`).
- **`{project-root}/_bmad/config.user.yaml`** — personal/gitignored settings: `user_name`, `communication_language`, plus any module variable marked `user_setting: true`.
- **`{project-root}/_bmad/module-help.csv`** — registers the four SynthPanel capabilities (`QP`, `RP`, `EP`, `RB`) for the help system.

Both merge scripts use an anti-zombie pattern — existing entries for module `spb` are removed before writing fresh ones, so stale values never persist.

`{project-root}` is a **literal token** in config values — never substitute it with an actual path. It signals to the consuming LLM that the value is relative to the project root, not the skill root.

## On Activation

1. Read `./assets/module.yaml` for module metadata and variable definitions (`code: spb` is the module identifier).
2. Check if `{project-root}/_bmad/config.yaml` exists — if a `spb:` section is already present, inform the user this is an update.
3. Check for legacy per-module config at `{project-root}/_bmad/spb/config.yaml` and `{project-root}/_bmad/core/config.yaml`. If either exists:
   - Fresh install (no `spb:` section yet): inform the user installer config was detected and will be consolidated.
   - Legacy migration (`spb:` section already present): inform the user legacy values will be used as fallback defaults.
   - In both cases, per-module config files and directories are cleaned up after setup.
4. Verify the SynthPanel MCP server is reachable. If the user has not yet configured a SynthPanel MCP host, point them at `design-notes/mcp-host-setup.md` (DESIGN-5) and continue setup — the pack installs cleanly without the server, but `convene-*` workflows will fail at preflight until the server is up.

If the user provides arguments (`accept all defaults`, `--headless`, or inline values), map any provided values to config keys, use defaults for the rest, and skip interactive prompting. Still display the full confirmation summary at the end.

## Collect Configuration

Show defaults in brackets. Present all values together so the user can respond once with only the values they want to change. Never tell the user to "press enter" or "leave blank" — in a chat interface they must type something to respond.

**Default priority** (highest wins): existing new config values > legacy config values > `./assets/module.yaml` defaults.

**Core config** (only if no core keys exist yet): `user_name` (default: BMad), `communication_language` and `document_output_language` (default: English — single language question, both keys get the same answer), `output_folder` (default: `{project-root}/_bmad-output`).

**Module config** (read each `prompt`-bearing variable from `./assets/module.yaml`):
- `verdict_output_path` — where to save raw `panel_verdict.json` artifacts.
- `readback_card_path` — where to save rendered markdown verdict cards.
- `mcp_server_url` — SynthPanel MCP host URL or `stdio` for a local server.

## Write Files

Write a temp JSON file with collected answers structured as `{"core": {...}, "module": {...}}` (omit `core` if it already exists). Then run both scripts in parallel — they write to different files:

```bash
python3 ./scripts/merge-config.py --config-path "{project-root}/_bmad/config.yaml" --user-config-path "{project-root}/_bmad/config.user.yaml" --module-yaml ./assets/module.yaml --answers {temp-file} --legacy-dir "{project-root}/_bmad"
python3 ./scripts/merge-help-csv.py --target "{project-root}/_bmad/module-help.csv" --source ./assets/module-help.csv --legacy-dir "{project-root}/_bmad" --module-code spb
```

Both scripts emit JSON to stdout. If either exits non-zero, surface the error and stop. Check `legacy_configs_deleted` and `legacy_csvs_deleted` in the output to confirm cleanup.

Run `./scripts/merge-config.py --help` or `./scripts/merge-help-csv.py --help` for full usage.

## Create Output Directories

After writing config, create configured output directories. Resolve `{project-root}` to the actual project root for filesystem operations only; the paths stored in the config files keep the literal `{project-root}` token. Create:

- `output_folder` (core)
- `verdict_output_path` (module)
- `readback_card_path` (module)

Use `mkdir -p` for the full path.

## Cleanup Legacy Directories

After both merge scripts complete successfully, remove installer package directories. Skills are already installed at `.claude/skills/` — `_bmad/` should only contain config files.

```bash
python3 ./scripts/cleanup-legacy.py --bmad-dir "{project-root}/_bmad" --module-code spb --also-remove _config --skills-dir "{project-root}/.claude/skills"
```

The script verifies each legacy skill exists at `.claude/skills/` before removing anything. Idempotent on missing directories.

## Confirm

Use the merge-script JSON output to display: config values written (root vs `spb:` section), user settings written to `config.user.yaml`, help entries added (4 rows: `QP`, `RP`, `EP`, `RB`), fresh install vs update, legacy cleanup count. Then display the `module_greeting` from `./assets/module.yaml`.

## Outcome

Once `user_name` and `communication_language` are known, address the user by their configured name and communicate in their configured `communication_language` for the remainder of the session. Mention that Quinn (the Panel Facilitator) is now available — the user can describe a stuck decision or invoke `convene-panel` / `convene-quick-poll` directly.
