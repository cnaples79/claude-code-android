# Changelog

## [1.1.0] — 2026-03-19

### Major UX Overhaul

- Added Prerequisites section with F-Droid/Termux installation walkthrough
- Added "Choose Your Path" decision point at top of INSTALL.md
- Added "What to Do First" orientation section after Quick Start
- Path B rewritten with exact verified sequence (every step tested on fresh devices)
- Device table redesigned with feature columns and "Last Verified" dates
- Three devices verified: Samsung Galaxy S26 Ultra, Google Pixel 10 Pro, Samsung Galaxy S23+
- Added `CLAUDE_CODE_TMPDIR` as documented alternative to proot
- Native installer note clarified (doesn't work in native Termux, works in Path B)

### Bug Fixes

- Fixed kernel prerequisite excluding Android 14/15 users (was 6.12.x, now varies)
- Fixed "Problem 3 is Android 16-specific" (Node v24 hang affects all ARM64)
- Aligned curl across Quick Start, INSTALL.md, and install.sh
- Stripped private repo paths from verification scripts
- Fixed `which` to `command -v` for portability
- Fixed CODE_OF_CONDUCT contact method
- Fixed install.sh shebang for desktop inspection
- Removed stale FD limit claims (~1024 → varies by device)
- Fixed duplicate EMFILE "Cause" paragraph with contradictory numbers

### Verification

- Added `tests/verify-claims.sh` — automated verification of all documentation claims
- Verification results linked from device table and INSTALL.md

## [1.0.0] — 2026-03-19

### First Stable Release

- One-command installer (`install.sh`) — installs packages, Claude Code, ripgrep fix, and shell alias in one pass
- Terminal screenshot proving Claude Code runs natively on Samsung Galaxy S26 Ultra
- GitHub release v1.0.0

## [0.4.0] — 2026-03-19

### Repo Quality Pass

- Fixed false proot-distro claim in CONSTITUTION-TEMPLATE.md (was shipping wrong info to every user who copied it)
- Added "Keeping It Running" section to INSTALL.md (update, ripgrep re-fix, uninstall)
- Added shields.io badges to README (license, Android version, Node.js version, Claude Code version, last verified date)
- Populated device compatibility table with verified device (Samsung Galaxy S26 Ultra) and common devices as "untested"
- Added `disable-model-invocation: true` to `/fix-ripgrep` skill (prevents auto-invocation of a skill that installs packages)

## [0.3.0] — 2026-03-19

### Documentation Correction — proot-distro Works on Android 16

Previous documentation incorrectly stated that proot-distro was broken on Android 16 due to a "kernel-level restriction" with "no fix inside the guest distro." This was wrong.

**What actually happened:** A TCGETS2 ioctl bug in proot broke stdout in guest distros using glibc 2.41+. This was fixed in proot 5.1.107-66 (October 2025). Current proot versions (5.1.107-70+) handle guest distros correctly on kernel 6.12.

**What changed:**
- Corrected all false claims about proot-distro being broken
- Added Path B installation guide (proot-distro Ubuntu) as a valid alternative
- Documented native installer (`curl -fsSL https://claude.ai/install.sh | bash`) for Path B
- Updated TROUBLESHOOTING.md proot-distro entry with current status and upgrade instructions
- Verified: Ubuntu 25.10 installs, Claude Code 2.1.79 runs via native installer inside guest

### Two Installation Paths

Users now have a documented choice:
- **Path A (Native Termux):** 4 commands, ~2 min, lighter — recommended for most users
- **Path B (proot-distro Ubuntu):** Full Linux env, no /tmp workaround needed, native installer — for users who want a complete Linux environment

## [0.2.0] — 2026-03-18

### Skills — First Android/Termux Skills in the Ecosystem

- `/doctor` — full environment diagnostic (Node, proot, TMPDIR, ripgrep, phantom killer, storage)
- `/fix-ripgrep` — detect and fix missing arm64-android ripgrep binary (Grep/Glob ENOENT fix)
- `termux-safe` — auto-loaded constraints preventing sudo, wrong paths, silent failures

### Improvements

- Broadened support from Android 16 to Android 14+
- Added 4 new troubleshooting entries: OAuth, voice mode, ripgrep ENOENT, hooks/platform detection
- Added upstream issues table to TROUBLESHOOTING.md
- Added table of contents to TROUBLESHOOTING.md
- Fixed logo identity leak (pilgrim-logo.jpg → logo.jpg)
- Added CONTRIBUTING.md, CODE_OF_CONDUCT.md, SECURITY.md, CHANGELOG.md, PR template

## [0.1.0] — 2026-03-18

### Initial Public Release

- README with 4-command Quick Start
- Full step-by-step install guide (INSTALL.md)
- Troubleshooting reference with 13 entries (TROUBLESHOOTING.md)
- CLAUDE.md constitution template for Android/Termux (CONSTITUTION-TEMPLATE.md)
- Issue templates for bug reports and device compatibility
- MIT license

### Tested On
- Samsung Galaxy S26 Ultra, Android 16, kernel 6.12.x, Node.js v25.8.1
