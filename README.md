# hskue

`hskue` is a small shell wrapper around `hs submit` that snapshots a job script before queueing it.

This helps preserve the exact script that was submitted, even if the original file changes later.

## Install with uv

From this checkout:

```bash
uv tool install .
```

While iterating on the script locally, use editable mode instead:

```bash
uv tool install --editable .
```

That installs the `hskue` command onto your `PATH`, so you can run:

```bash
hskue -- ./train.sh
```

## What It Does

When you submit a job with `hskue`, it:

1. Resolves the job script to an absolute path.
2. Copies that script into a history directory with a timestamped snapshot folder.
3. Writes a small launcher script that runs from the original submit-time working directory.
4. Stores basic submission metadata.
5. Calls `hs submit` with your original submit arguments and job arguments.

## Requirements

- `bash`
- `hs` (HyperShell) available on `PATH`, or passed via `--hs-bin` / `HS_BIN`
- `sha256sum`
- Either `realpath` or `readlink -f`

## Usage

```bash
hskue [wrapper args...] [hs submit args...] -- path/to/job.sh [job args...]
```

Examples:

```bash
hskue -- ./train.sh
hskue --history-dir ~/.local/state/hskue/history -- ./train.sh
hskue -t model:vit -t gpu:a100 -- ./train.sh --epochs 50
HS_SUBMITTED_HISTORY_DIR=~/.local/state/hskue/history hskue -- ./train.sh
```

## Wrapper Arguments

- `--history-dir DIR` overrides the snapshot directory.
- `--hs-bin PATH` overrides the `hs` executable.
- `-h`, `--help` prints help text.

Everything before `--` that is not a wrapper argument is passed through to `hs submit`.

Everything after `--` is treated as:

- the job script path
- followed by any job arguments passed to that script

## History Directory

By default, snapshots are stored in a fixed per-user directory:

```text
~/.local/state/hskue/history
```

You can override that with:

- `--history-dir`
- `HS_SUBMITTED_HISTORY_DIR`

Each submission creates a folder like:

```text
~/.local/state/hskue/history/20260319T153000_train_ab12cd34ef56/
```

That folder contains:

- the copied job script
- `launch.sh`, which runs the snapshot from the original working directory
- `metadata.txt`, which records submission details
- old snapshot folders are pruned automatically so only the newest 100 remain

## Notes

- The queued job runs from the working directory where `hskue` was invoked.
- If the original script has a shebang, that interpreter is used for the snapshot.
- If the script does not have a shebang, the launcher falls back to `bash`.
- The snapshot preserves file mode and timestamps from the original script.
- If `HOME` is unavailable, `hskue` falls back to `./hs_submitted_history`.
- `hskue` prints a warning if it cannot detect a local `hs cluster`, `hs server`, or `hs client` process before submission.

## Repository Contents

- [pyproject.toml](/home/studschaeffer/src/hs_queue/pyproject.toml)
- [hskue](/home/studschaeffer/src/hs_queue/hskue)
- [.gitignore](/home/studschaeffer/src/hs_queue/.gitignore)
