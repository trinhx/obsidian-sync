# obsidian-sync

Mirror project folders to and from an [Obsidian](https://obsidian.md) vault using `rsync`.

Useful for keeping working files (notes, tunes, drive dumps, etc.) in your project repo while also surfacing them inside an Obsidian vault — without copy-paste drift.

## Install

```sh
git clone https://github.com/<you>/obsidian-sync.git
ln -s "$PWD/obsidian-sync/obsidian-sync" ~/.local/bin/obsidian-sync
```

Requires `bash`, `rsync`, and `git` (for the `commit` command).

## Quick start

```sh
cd ~/Repos/my-project
obsidian-sync init    # pick folders to sync, confirm vault destination
obsidian-sync push    # local -> Obsidian
obsidian-sync pull    # Obsidian -> local
```

On first run, `obsidian-sync` scans `$HOME` for Obsidian vaults (directories containing `.obsidian/`) and writes the chosen base path to `~/.config/obsidian-sync/config`.

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
SYNC_SRCS=("drive" "tunes")     # folders relative to project root; "" means project root
SYNC_DEST_BASE="my-project/"    # destination base, relative to OBSIDIAN_BASE
```

Each entry maps `./<src>/` ↔ `<OBSIDIAN_BASE>/<SYNC_DEST_BASE>/<src>/`.

### Global (`~/.config/obsidian-sync/config`)

```sh
OBSIDIAN_BASE=/home/you/Documents/MyVault/Agent-Workspace/Projects
```

Edit this file to point at a different vault base.

## Example

```sh
cd ~/Repos/honda-tune
obsidian-sync init        # pick "1,2" -> drive + tunes; accept default base
obsidian-sync push
# mirrors honda-tune/drive/ -> <vault>/honda-tune/drive/
# mirrors honda-tune/tunes/ -> <vault>/honda-tune/tunes/
```
