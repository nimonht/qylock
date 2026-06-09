# Arch Linux packaging for qylock

This directory is a self-contained Arch package for **qylock** — the SDDM +
Quickshell lockscreen theme collection. It is laid out so it can be dropped
straight into the AUR (`PKGBUILD` + `.SRCINFO` + helper files in one folder).

## What it installs

| Path | Contents |
|---|---|
| `/usr/share/qylock/themes/` | All theme data (one copy, shared) |
| `/usr/share/qylock/quickshell/` | Quickshell lockscreen runtime |
| `/usr/share/qylock/fonts.manifest` | Font list used by `qylock-fetch-fonts` |
| `/usr/share/sddm/themes/qylock-*` | One **symlink** per theme so SDDM can find it (no duplication) |
| `/usr/bin/qylock-lock` | Launch the Quickshell lockscreen |
| `/usr/bin/qylock-sddm` | Select / list / reset the SDDM login theme |
| `/usr/bin/qylock-fetch-fonts` | Fetch or install per-theme fonts |

Theme data is installed **once** under `/usr/share/qylock/themes`; every SDDM
theme entry is a symlink into that tree, so the ~560 MB of wallpapers is never
duplicated between the SDDM and Quickshell halves.

## Install with paru

```sh
# From this directory (local build):
makepkg -si

# Or, once published to the AUR:
paru -S qylock
```

The package only hard-depends on the Qt6 QML runtime needed to render a theme.
Everything else is an `optdepend` — install what you actually use:

```sh
# Login screen themes
sudo pacman -S sddm
# Quickshell lockscreen
sudo pacman -S quickshell
# Video wallpapers
sudo pacman -S qt6-multimedia qt6-multimedia-ffmpeg \
               gst-plugins-base gst-plugins-good gst-plugins-bad gst-plugins-ugly
```

## Usage

```sh
# Login screen (SDDM)
qylock-sddm list
qylock-sddm set nier-automata        # writes /etc/sddm.conf.d/zz-qylock.conf
qylock-sddm set clockwork/orbital

# Lockscreen (Quickshell) — bind in your WM, e.g. Hyprland:
#   bind = SUPER, L, exec, qylock-lock
qylock-lock                          # saved/default theme
qylock-lock clockwork/tape           # a specific theme
```

## Fonts

A handful of themes look best with a font that **cannot be redistributed**
(proprietary licensing). `qylock-fetch-fonts` automates getting them into place
without ever bundling or hot-linking pirated files:

```sh
qylock-fetch-fonts --list                      # what's still missing
qylock-fetch-fonts                             # auto-handle what it can
qylock-fetch-fonts --install nier-automata "/path/to/FOT-Rodin Pro DB.otf"
qylock-fetch-fonts --url minecraft https://example.com/minecraft.ttf
```

What it does automatically:

* **osumania** — copies the Torus font already shipped with the `osu` theme.
* Anything with a `source` URL in the manifest is downloaded and verified.

What needs a font you legally own (proprietary — see the manifest for each):
`nier-automata`, `terraria`, `Genshin`, `minecraft`, `star-rail`, `windows_7`.
Supply them with `--install`, a per-user manifest (`~/.config/qylock/fonts.manifest`),
or `QYLOCK_FONT_URL__<theme>` env vars.

Fonts land in `/usr/share/qylock/themes/<theme>/font/`, which both the SDDM
greeter and the Quickshell lockscreen read.

## Clean uninstall (bloat-free)

```sh
paru -Rns qylock     # or: sudo pacman -Rns qylock
```

pacman removes every packaged file. The `qylock.install` hook then removes the
things that are created *after* install and would otherwise linger:

* `/usr/share/qylock` — including any fonts fetched by `qylock-fetch-fonts`
* `/etc/sddm.conf.d/zz-qylock.conf` — qylock's dedicated SDDM drop-in

Per-user state is left in place; remove it yourself if you want:

```sh
rm -rf ~/.config/qylock
```

Because qylock writes its SDDM choice to its **own** drop-in file rather than
editing a shared config, uninstalling never leaves a dangling `Current=` that
would point SDDM at a missing theme.

## Toward the official repositories

This PKGBUILD already follows Arch packaging standards (SPDX `license`,
`arch=any`, split `depends`/`optdepends`, an `.install` hook, real checksums for
the bundled helpers, `namcap`-clean layout). Two things are needed before an
`[extra]` submission, which require upstream/maintainer action rather than code:

1. **A tagged release.** Replace the pinned `_commit` with a tagged tarball
   source and run `updpkgsums`:
   ```sh
   source=("$pkgname-$pkgver.tar.gz::$url/archive/refs/tags/v$pkgver.tar.gz")
   ```
2. **A Trusted User sponsor.** Official-repo inclusion goes through a TU; the
   usual path is to publish on the AUR first, gather users, then request a move.
