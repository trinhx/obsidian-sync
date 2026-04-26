# obsidian-sync

Mirror project folders to and from an [Obsidian](https://obsidian.md) vault using `rsync`.

Useful for keeping working files (notes, drafts, attachments, etc.) inside your project repo while also surfacing them inside an Obsidian vault — without copy-paste drift. Each project gets its own `.obsidian-sync` config, so you can mirror many projects into one vault.

## Requirements

| Tool                                | Why                                                                |
| ----------------------------------- | ------------------------------------------------------------------ |
| `bash` 4+                           | The script uses bash arrays and `read -e -i`.                      |
| `rsync`                             | Does the actual mirroring.                                         |
| `git`                               | Only needed for `obsidian-sync commit`. Optional otherwise.        |
| [Obsidian](https://obsidian.md)     | A vault somewhere under `$HOME` (any folder containing `.obsidian/`). |

Install the dependencies if you don't already have them:

```sh
# Debian / Ubuntu
sudo apt install bash rsync git

# Fedora / RHEL
sudo dnf install bash rsync git

# Arch
sudo pacman -S bash rsync git

# macOS (Homebrew) — macOS ships bash 3.2, so install a newer one
brew install bash rsync git
```

> **macOS note:** the system `/bin/bash` is 3.2 and won't work. After `brew install bash`, the script's `#!/bin/bash` shebang will still pick up the old one — either run it as `bash obsidian-sync ...`, or change the shebang to `#!/usr/bin/env bash` and ensure Homebrew's `bash` comes first on your `PATH`.

## Install

Clone the repo and symlink the script onto your `PATH`:

```sh
git clone https://github.com/trinhx/obsidian-sync.git ~/.local/share/obsidian-sync
mkdir -p ~/.local/bin
ln -s ~/.local/share/obsidian-sync/obsidian-sync ~/.local/bin/obsidian-sync
```

Make sure `~/.local/bin` is on your `PATH`. If it isn't, add this to your `~/.bashrc` or `~/.zshrc`:

```sh
export PATH="$HOME/.local/bin:$PATH"
```

Reload your shell, then verify:

```sh
obsidian-sync help
```

### Alternative: no symlink

If you'd rather not symlink, you can also just run the script directly:

```sh
git clone https://github.com/trinhx/obsidian-sync.git
cd obsidian-sync
./obsidian-sync help
```

### Updating

```sh
cd ~/.local/share/obsidian-sync && git pull
```

### Uninstall

```sh
rm ~/.local/bin/obsidian-sync
rm -rf ~/.local/share/obsidian-sync
rm -rf ~/.config/obsidian-sync     # global config (optional)
```

`.obsidian-sync` files inside your projects are left untouched — delete them per project if you no longer need them.

## Quick start

```sh
cd ~/Repos/my-project
obsidian-sync init    # pick folders to sync, confirm vault destination
obsidian-sync push    # local -> Obsidian
obsidian-sync pull    # Obsidian -> local
```

On first run, `obsidian-sync` scans `$HOME` for Obsidian vaults (directories containing `.obsidian/`) and writes the chosen base path to `~/.config/obsidian-sync/config`. You can also enter a custom path if your vault lives elsewhere.

## Commands

| Command  | Description                                                                   |
| -------- | ----------------------------------------------------------------------------- |
| `init`   | Interactively create a `.obsidian-sync` config in the current project.        |
| `status` | Show `diff -rq` between local and vault for each configured source.           |
| `pull`   | Sync vault → local (`rsync --delete`).                                        |
| `push`   | Sync local → vault (`rsync --delete`).                                        |
| `commit` | Pull from vault, then `git commit && git push` in BOTH the project and vault. |
| `help`   | Print usage.                                                                  |

> **Note:** `pull` and `push` use `rsync --delete` — files missing on the source side are deleted on the destination side. Run `status` first if unsure.

## Configuration

### Per-project (`./.obsidian-sync`)

Written by `init`:

```sh
SYNC_SRCS=("notes" "drafts")    # folders relative to project root; "" means project root
SYNC_DEST_BASE="my-project/"    # destination base, relative to OBSIDIAN_BASE
```

Each entry maps `./<src>/` ↔ `<OBSIDIAN_BASE>/<SYNC_DEST_BASE>/<src>/`.

Commit `.obsidian-sync` to your repo if you want the same layout on every machine — paths inside it are relative to the project root and the vault base, so there's nothing host-specific in it.

### Global (`~/.config/obsidian-sync/config`)

```sh
OBSIDIAN_BASE=/path/to/your/ObsidianVault/Projects
```

This file is created on first run. Edit it to point at a different vault or a different subdirectory inside the vault.

## `commit` command

`obsidian-sync commit` is a convenience workflow for the common case where both the project and the Obsidian vault are themselves git repos:

1. Pull the latest from the vault into the project (`rsync --delete`).
2. `git add . && git commit && git push` in the project repo.
3. `git add . && git commit && git push` in the vault repo (the parent of `OBSIDIAN_BASE`).

If your vault is not a git repo, just use `pull`/`push` instead.

## Example

```sh
cd ~/Repos/my-project
obsidian-sync init        # pick "1,2" -> notes + drafts; accept default base
obsidian-sync push
# mirrors my-project/notes/  -> <vault>/my-project/notes/
# mirrors my-project/drafts/ -> <vault>/my-project/drafts/
```

## License

MIT
