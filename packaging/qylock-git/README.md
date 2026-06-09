# qylock-git — Arch (AUR) package

This folder **packages** [qylock](https://github.com/Darkkal44/qylock) by
Darkkal44 for Arch Linux. It does not fork or modify the project — the theme
data is pulled straight from Darkkal's upstream repo at build time, so all
credit and upstream changes stay with him. The only things here are the
packaging files (PKGBUILD, helpers, install hook).

It's a VCS (`-git`) package because upstream has no tagged releases: it builds
the latest commit.

## Install

```sh
cd packaging/qylock-git
makepkg -si
```

(Once it's published to the AUR: `paru -S qylock-git`.)

## What it installs

| Path | Contents |
|---|---|
| `/usr/share/qylock/themes/` | All theme data (one copy, shared) |
| `/usr/share/qylock/quickshell/` | Quickshell lockscreen runtime |
| `/usr/share/sddm/themes/qylock-*` | One **symlink** per theme so SDDM finds it (no duplication) |
| `/usr/bin/qylock-lock` | Launch the Quickshell lockscreen |
| `/usr/bin/qylock-sddm` | Select / list / reset the SDDM login theme |
| `/usr/bin/qylock-fetch-fonts` | Fetch or install the per-theme fonts |

## Usage

```sh
qylock-sddm set nier-automata        # pick the login (SDDM) theme
qylock-lock clockwork/tape           # run the lockscreen (bind in your WM)
qylock-fetch-fonts --list            # see / install the missing fonts
```

## Fonts

A few themes need a font that cannot be redistributed. `qylock-fetch-fonts`
auto-copies the ones it can (e.g. osumania reuses osu's font) and tells you how
to supply the rest (`--install <theme> <file>`, a URL, or a per-user manifest).
Fonts land in `/usr/share/qylock/themes/<theme>/font/`.

## Update

* Built locally with `makepkg`: re-run `makepkg -si` to grab the latest commit.
* Installed from the AUR with paru: `paru -Syu` rebuilds it automatically when
  upstream changes (paru's devel tracking only works for AUR-installed packages).

## Uninstall (bloat-free)

```sh
paru -Rns qylock-git        # or: sudo pacman -Rns qylock-git
```

pacman removes every packaged file; the install hook then clears the bits made
*after* install — fetched fonts under `/usr/share/qylock` and the dedicated
`/etc/sddm.conf.d/zz-qylock.conf` drop-in — so nothing lingers. Per-user state,
if any, is left for you to remove: `rm -rf ~/.config/qylock`.

## Publishing to the AUR

The folder is already in AUR layout (`PKGBUILD` + `.SRCINFO` + helpers). To
publish:

```sh
git clone ssh://aur@aur.archlinux.org/qylock-git.git
cp packaging/qylock-git/{PKGBUILD,.SRCINFO,qylock.install,qylock-lock,qylock-sddm,qylock-fetch-fonts,fonts.manifest} qylock-git/
cd qylock-git
# set your "# Maintainer:" line, then:
makepkg --printsrcinfo > .SRCINFO
git add -A && git commit -m "Initial import" && git push
```
