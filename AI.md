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
