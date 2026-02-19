# filename-standards

Two Bash scripts:

* `rename_safe.sh` renames entries by replacing spaces with `_` (optionally recursive / directories / lowercase) and writes logs.
* `undo_rename.sh` reverses successful renames using the machine log (`.bin`) produced by `rename_safe.sh`.

## Requirements

* Bash 4+ (for lowercase conversion when `--lowercase` is used)
* `find`, `mv` (standard on Linux)

## Install

```bash
chmod +x rename_safe.sh undo_rename.sh
```

Optional: run from anywhere by symlinking into `~/.local/bin`:

```bash
mkdir -p ~/.local/bin
ln -sf "$PWD/rename_safe.sh" ~/.local/bin/rename-safe
ln -sf "$PWD/undo_rename.sh" ~/.local/bin/undo-rename
export PATH="$HOME/.local/bin:$PATH"
```

## rename_safe.sh

### What it does

* Replaces spaces in names with `_`
* Collapses multiple underscores (`__` -> `_`)
* Dry-run by default (no changes unless `--apply` is used)
* Non-recursive by default
* Does not rename directories unless `--dirs` is used
* Writes:

  * a human log: `rename_YYYYMMDD_HHMMSS.log`
  * a machine log for undo: `rename_YYYYMMDD_HHMMSS.bin` (NUL-delimited)

### Usage

Dry run (current directory only, files only):

```bash
./rename_safe.sh
```

Apply the same operation:

```bash
./rename_safe.sh --apply
```

Recursive (files only):

```bash
./rename_safe.sh --recursive
./rename_safe.sh --recursive --apply
```

Recursive including directories:

```bash
./rename_safe.sh --recursive --dirs
./rename_safe.sh --recursive --dirs --apply
```

Enable lowercasing (off by default):

```bash
./rename_safe.sh --recursive --lowercase
./rename_safe.sh --recursive --lowercase --apply
```

### Flags

* `--apply`
  Perform the renames. Without this flag the script prints and logs the planned changes.
* `--recursive`
  Traverse subdirectories. Without this flag only the current directory is processed.
* `--dirs`
  Also rename directories. Without this flag only non-directories (files, symlinks, etc.) are renamed.
* `--lowercase`
  Convert the basename to lowercase after space replacement. Off by default.
* `--log-dir DIR`
  Directory to write logs into (default: `.`).
* `--log-basename STR`
  Log base name (default: `rename_YYYYMMDD_HHMMSS`).
* `--include GLOB` (repeatable)
  Only consider paths (relative to the current directory) matching this Bash glob.
* `--exclude GLOB` (repeatable)
  Skip paths (relative to the current directory) matching this Bash glob.
* `--git`
  Prefer `git mv` when inside a git repo (falls back to `mv` if needed).

### Logs

After a run you will have:

* `rename_*.log` (human-readable)
* `rename_*.bin` (machine log for undo)

To inspect the `.bin` file:

```bash
tr '\0' '\n' < rename_*.bin
```

## undo_rename.sh

### What it does

* Reads the `.bin` machine log from `rename_safe.sh`
* Undoes only successful renames (`OK` / `OK_GIT`)
* Runs in reverse order (safe for directories)
* Dry-run by default

### Usage

Dry-run undo:

```bash
./undo_rename.sh rename_YYYYMMDD_HHMMSS.bin
```

Apply undo:

```bash
./undo_rename.sh --apply rename_YYYYMMDD_HHMMSS.bin
```

Prefer `git mv` during undo (optional):

```bash
./undo_rename.sh --apply --git rename_YYYYMMDD_HHMMSS.bin
```

### Flags

* `--apply`
  Perform the undo operations. Without this flag it prints and logs what would be undone.
* `--git`
  Prefer `git mv` when inside a git repo (falls back to `mv` if needed).

### Undo behavior

* Skips if the source-to-undo no longer exists.
* Skips if the destination already exists (does not overwrite).
* Writes an undo report log: `undo_YYYYMMDD_HHMMSS.log` in the current directory.

## Recommended workflow

1. Run `rename_safe.sh` without `--apply` and review the `PLAN` lines in the `.log`.
2. Re-run the same command with `--apply`.
3. If anything goes wrong, run `undo_rename.sh` on the corresponding `.bin` file.
