# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Userspace tools for AmneziaWG, a fork of WireGuard's `wireguard-tools` (git.zx2c4.com) maintained by the Amnezia VPN project. C (`-std=gnu99`), with bash `wg-quick` scripts and a Windows cross-build. Binaries install as **`awg`** and **`awg-quick`** (not `wg`/`wg-quick`) — this is the AmneziaWG branding.

## Build & check

All builds run from `src/`:

```bash
cd src
make                 # build awg
make check           # scan-build static analysis (run on C changes before done)
make install PREFIX=/usr DESTDIR=...
```

- Optional features auto-detect; force with `WITH_WGQUICK`, `WITH_BASHCOMPLETION`, `WITH_SYSTEMDUNITS` (yes/no).
- `DEBUG=yes` adds `-g`; `V=1` for verbose build output.
- Version is injected from `git describe` at build time — don't hardcode it.
- Windows: `build.cmd` cross-compiles x64/x86/arm64 via llvm-mingw (downloads toolchain + wireguard-nt).
- Fuzzers live in `src/fuzz/` (clang `-fsanitize=fuzzer,address`).

## Conventions

- **Track upstream closely.** Keep diffs from `wireguard-tools` minimal and preserve upstream structure/style. AmneziaWG protocol changes (e.g. S/H/I parameters) should be surgical additions, not rewrites.
- **Platform split.** IPC and uapi code is split per-platform (`ipc-linux.h`, `ipc-windows.h`, `ipc-openbsd.h`, `ipc-freebsd.h`, `uapi/{linux,freebsd,openbsd,haiku,windows}`). When touching these, mirror the existing pattern rather than adding `#ifdef` branches to a shared file.
- **SPDX headers** on every new source file (`GPL-2.0 OR MIT`, matching the file's siblings).
- **Commits/PRs:** conventional prefixes (`fix:`, `feat:`, `revert:`); feature branches merged via PR into `master`.

## Gotchas

- `addconf` and `syncconf` both dispatch to `setconf_main` with different internal semantics.
- AmneziaWG adds custom config/handshake parsing (e.g. `clean_special_handshake_line()`) on top of the upstream parser.
- No dependencies beyond a C compiler and libc.
