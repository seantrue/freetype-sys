# AI Contributions — freetype-sys (seantrue fork)

*About AI slop.* Fair enough: the patch this fork carries was written by a machine. Call it
instant ramen: fast, cheap, genuinely edible, and nobody should pretend it simmered on a stove
all afternoon. The honest answer is the record below: what the maintainer asked for, what the
model wrote, and what went wrong, including defects in AI-written code that building the
dependent projects on real Windows and Linux machines caught rather than luck. Bon appétit, but
mind the sodium content.

Tracks what AI helpers contributed to this fork versus the human maintainer.

## Human maintainer

freetype-sys itself is the PistonDevelopers project (223 commits since 2014-12-11; Sven Nilsen,
Bruce Mitchener, tomaka and others), none of it AI work. This fork exists only to carry a
Windows build fix for the maintainer's projects (yapocis-rs, impressionist-rs, impcraft), which
consume it through `[patch.crates-io]`. The maintainer chose the fork-and-patch approach and
decided against proposing the change upstream.

## Contributions

### 2026-11-09 — bundled libpng builds on ARM64 Windows (MSVC) (Claude Opus 5.5)

**What the maintainer asked for.** Build virtimp_com_app's desktop app natively on a Windows 11
ARM64 VM (UTM on the Mac), "until it builds and renders", then push this fix to the fork.

**Cause.** On every `arm`/`aarch64` target, `build.rs` compiles libpng's `arm/filter_neon.S`.
That file is GNU assembler, and under MSVC (`aarch64-pc-windows-msvc`) cc-rs produces no object
for it, so archiving `libpng.a` failed with `LNK1181: cannot open input file
'…-filter_neon.o'`. The file has code only when libpng uses its hand-written assembly
(`PNG_ARM_NEON_IMPLEMENTATION == 2`); `pngpriv.h` picks the intrinsics (`1`) whenever
`arm_neon.h` exists, as it does under MSVC, so the file is empty there anyway.

**What the AI did.** `build.rs` (~6 LOC): skip `filter_neon.S` when `CARGO_CFG_TARGET_ENV` is
`msvc`; every other ARM target still compiles it, unchanged.

**Verified.** On the ARM64 VM (MSVC 14.44, HostARM64), with this `build.rs` applied as a local
`[patch.crates-io]`: virtimp_com_app's `desktop/` built (`--release --features
virtimp-local/rustcpu`), its 18 unit tests passed, and `virtimp-desktop render` produced a
correct 1200×1600 Pen and Ink #1 of the test photograph on the rustcpu engine (35.7 s). Not
built on other ARM targets after the change; their path through `build.rs` is untouched.

### 2026-10-25 — bundled libpng finds zlib.h via DEP_Z_INCLUDE (Claude Opus 5.5)

**What the maintainer asked for.** Fix the Windows build of yapocis's `text` feature
(`freetype-rs` with `bundled`), which failed with
`pngstruct.h(30): fatal error C1083: Cannot open include file: 'zlib.h'`.

**Cause.** `build.rs` compiles the bundled libpng with `-I libz-sys/src/zlib`. That path exists
in a git checkout (`libz-sys` is a submodule) but not in the crates.io package, so the published
0.23.0 relies on a system `zlib.h`: present on Linux/macOS, absent under MSVC.

**What the AI did.** `build.rs` (~9 LOC): the libpng build takes its zlib include directories
from `DEP_Z_INCLUDE`, which `libz-sys` (`links = "z"`) exports, splitting on commas as libz-sys
does when it finds zlib via pkg-config/vcpkg; with no `DEP_Z_INCLUDE` it falls back to the system
include path. This also makes libpng compile against the headers of the zlib actually linked.

**Verified.** On the win11-dev VM (MSVC 14.44), `cargo build -p yapocis --features text,ocl` in
yapocis-rs, with this `build.rs` applied as a local `[patch.crates-io]`: freetype-sys and its
libpng compiled; the build then stopped on unrelated `c_long` type errors in yapocis itself.
