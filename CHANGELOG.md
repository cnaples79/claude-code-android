# Changelog

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
