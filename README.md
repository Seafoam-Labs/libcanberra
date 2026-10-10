# libcanberra for Devario

Seafoam Labs hosts libcanberra to provide a stable HTTPS source location for
building and distributing **Devario**. Devario's package infrastructure needs
reproducible access to dependencies even when the original upstream Git hosting
is unavailable or has certificate problems.

This is a distribution-maintained source mirror of the work of Lennart
Poettering and contributors. Seafoam Labs does not claim upstream ownership or
represent this snapshot as a new upstream release.

## Why this mirror exists

Upstream's Git hosting is gone, not merely misconfigured. Measured 2026-10-10:

- The certificate served by `git.0pointer.net` and `git.0pointer.de` was
  renewed on 2026-09-27 and no longer lists either `git.` hostname in its
  Subject Alternative Names, so every TLS client rejects it:
  `SSL: no alternative certificate subject name matches target hostname`.
- Behind that, the git service itself is gone. Every path on both hosts, over
  HTTPS and plain HTTP, answers `301 Moved Permanently` to `http://0pointer.net/`,
  so the smart-HTTP endpoint redirects to the project homepage instead of
  serving `info/refs`. The `git://` daemon on port 9418 accepts connections but
  serves no repository.

Disabling TLS verification does not help, because the redirect target is not a
git endpoint. Arch's own `libcanberra` recipe still names the dead URL and
cannot be built from a clean source cache either. The project page at
<https://0pointer.net/lennart/projects/libcanberra/> still answers and still
advertises the dead `git://git.0pointer.de/libcanberra` clone URL.

## Current source snapshot

The source on `upstream` matches the commit Arch pins:

```text
c0620e432650e81062c1967cc669829dbd29b310
```

- Upstream commit date: **9 November 2012**.
- Upstream commit subject: **gtk: Don't assume all GdkDisplays are GdkX11Displays: broadway/wayland**.
- Package version: **0.30+r2+gc0620e4**, two commits after upstream v0.30.
- The [upstream branch](https://github.com/Seafoam-Labs/libcanberra/tree/upstream)
  preserves that original commit and all 398 commits of its Git history,
  together with the upstream release tags `v0.1` through `v0.30`.
- `main` adds this README, the Devario PKGBUILD, its `.SRCINFO` metadata, and
  the two local files the recipe needs.

The history was recovered from the
[Distrotech mirror](https://github.com/Distrotech/libcanberra), described there
as a mirror of `git://git.0pointer.de/libcanberra`. Its vendor branch and its
two `distrotech-libcanberra-*` tags are not carried here. The
[openEuler-Networking mirror](https://github.com/openEuler-Networking/libcanberra)
was checked independently and reports the same commit and the same tree
`1630f35feb690378c6a20b8ec81ec6b183dda897`. Because the pinned commit ID is
itself a hash of its contents, agreement on that ID is what proves the snapshot
matches the source Arch built.

### No snapshot tag on the pinned commit

Unlike the libasyncns mirror, this repository deliberately places **no** tag on
`c0620e4`. The Devario recipe derives its version at build time:

```sh
git describe --tags | sed 's/^v//;s/[^-]*-g/r&/;s/-/+/g'
```

A tag sitting directly on the pinned commit would make `git describe` return
that tag alone, shortening the version from `0.30+r2+gc0620e4` to whatever the
tag expands to and silently changing the package version. The upstream `v0.30`
tag is the nearest one, so `git describe --tags` returns `v0.30-2-gc0620e4` and
the version stays correct. Keep it that way if you add tags here.

## What libcanberra does

libcanberra is a small implementation of the XDG Sound Theme Specification: it
plays a named sound event from the configured theme, optionally through a
desktop-specific backend. GTK+ applications load it through a GTK module so
that widget actions produce themed sounds. The Devario recipe builds the ALSA
and PulseAudio backends as loadable DSO plugins, and the PulseAudio client
backend is what a PipeWire-based desktop uses through PipeWire's PulseAudio
compatibility service.

## Building a Devario package

The root [PKGBUILD](PKGBUILD) builds **1:0.30+r2+gc0620e4-6** for x86_64. It
clones the source directly from this repository over HTTPS and checks out the
original upstream commit:

```text
git+https://github.com/Seafoam-Labs/libcanberra.git#commit=c0620e432650e81062c1967cc669829dbd29b310
```

The commit is pinned independently of changes to the packaging branch. Git
sources use `sha256sums=('SKIP')`; source selection is fixed by the full commit
ID, with TLS verification enabled.

The source cache and checkout are named `seafoam-libcanberra` so that an old
`libcanberra` source cache pointing at 0pointer is not reused. There is no
fallback to 0pointer or another source host.

Use an Arch-compatible `base-devel` environment with `libvorbis`, `libltdl`,
`alsa-lib`, `libpulse`, `tdb`, `sound-theme-freedesktop`, `gtk-doc`, `gtk3`,
and `git` installed:

```sh
makepkg
```

Shelly can review and build the same recipe:

```sh
shelly build --review-only --json ./PKGBUILD
shelly build --sync-deps ./PKGBUILD
```

This Git snapshot has no generated `configure` script, so `prepare()` runs
`./autogen.sh` and applies [libcanberra-multi-backend.patch](libcanberra-multi-backend.patch),
which lets more than one backend be selected at runtime
([FS#71341](https://bugs.archlinux.org/task/71341),
[fd.o#51662](https://bugs.freedesktop.org/show_bug.cgi?id=51662)).
[40-libcanberra-gtk-module.sh](40-libcanberra-gtk-module.sh) is installed into
`/etc/X11/xinit/xinitrc.d` and loads the GTK module only where a sound server is
actually reachable.

## License and attribution

libcanberra is copyright its upstream authors and is distributed under the
**GNU Lesser General Public License, version 2.1 or later**, as the upstream
[LGPL](LGPL) file records. The original source copyright notices and upstream
documentation are preserved. Seafoam Labs' role here is to host the source for
Devario.
