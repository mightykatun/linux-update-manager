# Linux Update Manager

<p align="center">
  <img src="assets/main-ui.png" alt="Main UI" height="300"><img src="assets/stats-ui.png" alt="Stats UI" height="300">
</p>

Linux Update Manager that lets you run all your system update commands from one YAML file.

You get a compact terminal UI, saved logs, run history, a detailed stats dashboard, ... If you struggle updating packages, language tools, and other local tools by hand, this gives you one place to save and manage that flow.

> [!WARNING]
> Your update steps live in `~/.config/update.yaml`, treat this file like arbitrary code, because the commands inside it runs through your shell. Keep it owned by your user and do not make it group- or world-writable.

## What It Does

- Runs package manager, tool updates, or any build/update command from one YAML config.
- Skips steps when required tools/conditions are met
- Saves per-command logs in `~/.update/logs`.
- Stores run history in `~/.update/history.jsonl`.
- Shows before and after stats from command output (e.g. `old version` -> `new version`)
- Supports dry runs and filters to only run one update.
- Includes a JSON Schema for editor validation and completion.

## Requirements

- Python 3.10 or newer.
- `PyYAML` and `rich`/`textual`.

Install the Python packages with your preferred tool. For example:

```sh
python -m pip install -r requirements.txt
```

## Install

```sh
install -m 755 update ~/.local/bin/update
mkdir -p ~/.config
cp examples/update.yaml ~/.config/update.yaml
```

Then edit `~/.config/update.yaml`

## Usage

```sh
update                    # run selected update steps
update -n                 # dry run
update -c                 # readiness check
update -s                 # stats

# filters
update --only "apt update"
update --skip "cargo update"
update --section APT

# testing
update --self-test
```

## Config

Start with `examples/update.yaml`.

The schema is in `schema/update.schema.json`. The example config links to it like this:

```yaml
# yaml-language-server: $schema=../schema/update.schema.json
```

A step looks like this:

```yaml
Section Name:
  step name:
    run: command to execute
    needs:
      - required-executable-or-path
    skip_if: shell condition that skips when it exits 0
    stat:
      after:
        changed: command that prints a number or short value
```

`run` can be one command or a list of commands.

`needs` is optional. If you leave it out, the updater tries to infer the required command from `run`.

`stat.version` cannot be used with `stat.before` or `stat.after`.

Use `$UPDATE_TMPDIR` when a command needs a scratch file that another stat command reads during the same run.

## State

Persistent state is stored in `~/.update`:

```text
~/.update/history.jsonl
~/.update/logs/
~/.update/update.lock
```

The saved logs help when a step fails.

`$UPDATE_TMPDIR` is different. It is temporary scratch space for one run, and it gets removed after that run finishes.

## Security

This tool runs configured commands through a shell. Treat `update.yaml` as executable code.

The script rejects configs that are not owned by the current user. It also rejects configs that are writable by group or others.
