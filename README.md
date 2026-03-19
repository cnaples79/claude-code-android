# Claude Code on Android

<p align="center">
  <img src="logo.jpg" alt="Claude Code on Android" width="200">
</p>

<p align="center">
  <img src="screenshot.jpg" alt="Claude Code running on Samsung Galaxy S26 Ultra" width="300">
</p>

<p align="center">
  <strong>Run Claude Code natively on Android — no root, no emulator, no cloud VM.</strong>
</p>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-yellow.svg" alt="License: MIT"></a>
  <img src="https://img.shields.io/badge/Android-14%2B-brightgreen.svg" alt="Android 14+">
  <img src="https://img.shields.io/badge/Node.js-v25%2B-green.svg" alt="Node.js v25+">
  <img src="https://img.shields.io/badge/Verified-Claude%20Code%202.1.79-blue.svg" alt="Verified with Claude Code 2.1.79">
  <img src="https://img.shields.io/badge/Last%20Verified-March%202026-lightgrey.svg" alt="Last Verified March 2026">
</p>

<p align="center">
  <a href="INSTALL.md">Install Guide</a> · <a href="TROUBLESHOOTING.md">Troubleshooting</a> · <a href="CONSTITUTION-TEMPLATE.md">CLAUDE.md Template</a>
</p>

---

## Prerequisites

You need **Termux** installed from **F-Droid** (not the Play Store — the Play Store version is outdated and won't work).

1. Download F-Droid from [f-droid.org](https://f-droid.org/en/)
2. Open the downloaded APK — Android will block it. Go to Settings → allow "install unknown apps" from your browser
3. **Security note:** After installing F-Droid, go back to Settings and disable "install unknown apps" from your browser. Keep it enabled only for F-Droid itself (F-Droid needs it to install apps)
4. Open F-Droid, search for **Termux**, install it
5. Android may warn "unsafe app — built for an older version." Tap **More details → Install anyway**. This is safe — Termux targets an older API level for broader compatibility
6. Open Termux

> **Already have Termux from F-Droid?** Skip to Quick Start.

---

## Quick Start

Four commands. Termux open. Go.

```bash
pkg install nodejs git curl proot ripgrep -y
export TMPDIR=$PREFIX/tmp   # Critical: npm fails silently without this
npm install -g @anthropic-ai/claude-code
proot -b $PREFIX/tmp:/tmp claude
```

That's it. Claude Code is running on your phone.

> **Scripted install:** If you're on a desktop terminal or can copy-paste from a browser, there's also a [one-command installer](install.sh) (`curl | bash`). But the four commands above are designed to be typed on a phone keyboard — no long URLs required.

> **Note:** The Quick Start commands work for this session. Add TMPDIR to your .bashrc (shown below) to make it permanent.

### What to Do First

- **Navigate to a project directory** before launching, or create one: `mkdir ~/myproject && cd ~/myproject`
- Claude Code works on files in your current directory
- Type `/help` inside Claude Code to see what it can do
- Run `/doctor` to verify your setup (after installing skills — see below)

Add this to `~/.bashrc` so it sticks:

```bash
echo 'export TMPDIR=$PREFIX/tmp' >> ~/.bashrc
echo "alias claude-android='proot -b \$PREFIX/tmp:/tmp claude'" >> ~/.bashrc
source ~/.bashrc
```

Now just type `claude-android`.

> **Requires:** [Termux from F-Droid](https://f-droid.org/en/packages/com.termux/) (not Play Store), Android 14+, Node.js v25+, Claude Max or Pro subscription.

---

## Why This Is Hard

Running Claude Code on Android requires solving three problems that have stopped others:

### 1. proot-distro works but is unnecessary overhead

A bug that broke proot-distro on kernel 6.12 was fixed in proot 5.1.107-66 (October 2025). Guest distros work correctly now. But a full guest OS is unnecessary for Claude Code — it only needs a writable `/tmp`, which a single proot bind mount provides. The Quick Start uses the lighter native Termux approach. For a full Linux environment, see [INSTALL.md — Path B](INSTALL.md#path-b-proot-distro-ubuntu).

### 2. /tmp doesn't exist

Claude Code hardcodes `/tmp` for sockets and IPC. On Android, `/tmp` isn't writable. Without it, Claude Code fails silently — no error, no crash log, just nothing. The fix: `proot -b $PREFIX/tmp:/tmp` remaps the path at the syscall level. No root required.

### 3. Node.js v24 hangs on ARM64

Node.js v24 hangs on startup under Termux on ARM64. The cause is unclear but upgrading to v25+ fixes it. Termux ships v25 by default now, so fresh installs avoid this.

---

## What's In This Repo

| File | What It Is |
|------|-----------|
| **[install.sh](install.sh)** | One-command installer — packages, Claude Code, ripgrep fix, shell config |
| **[INSTALL.md](INSTALL.md)** | Complete step-by-step install guide with Path A (native) and Path B (proot-distro) |
| **[TROUBLESHOOTING.md](TROUBLESHOOTING.md)** | Common failures with symptoms, causes, and fixes |
| **[CONSTITUTION-TEMPLATE.md](CONSTITUTION-TEMPLATE.md)** | A CLAUDE.md template for giving Claude Code persistent rules and identity on Android |
| **[CHANGELOG.md](CHANGELOG.md)** | Version history |
| **[.claude/skills/](.claude/skills/)** | Android-specific Claude Code skills (/doctor, /fix-ripgrep, termux-safe) |

---

## Known Constraints

Running on a phone means real limits. Know them upfront:

| Constraint | Impact | Workaround |
|-----------|--------|-----------|
| No root | No `sudo`, no ports below 1024 | Use ports 1024+, skip anything that needs root |
| No systemd | No services, no daemons the normal way | Use `crond` or shell scripts for persistence |
| ~512MB Node.js heap | Large datasets must stream | Don't buffer — stream and process incrementally |
| File descriptor limits | Heavy I/O can hit EMFILE on some devices | Limit concurrent processes |
| Phantom process killer | Android kills excess background processes | Use `tmux`, limit to 2-3 background processes |
| /tmp is volatile | proot crash = mount gone | Never store persistent state in /tmp |

See [TROUBLESHOOTING.md](TROUBLESHOOTING.md) for detailed fixes.

---

## Device Compatibility

Verified working on:

| Device | Android | Path A | Path B | Grep/Glob | Last Verified |
|--------|---------|--------|--------|-----------|---------------|
| Samsung Galaxy S26 Ultra | 16 | Works | Works | Symlink fix | 2026-03-19 |
| Google Pixel 10 Pro | 16 | Works | Works | Symlink fix | 2026-03-19 |
| Samsung Galaxy S23+ | 15 | Untested | Works | Not needed (Path B) | 2026-03-19 |
| Samsung Galaxy S24/S25 | 15-16 | Untested | Untested | — | — |
| Google Pixel 8/9 | 15-16 | Untested | Untested | — | — |
| OnePlus 12/13 | 14-15 | Untested | Untested | — | — |

Expected to work on any aarch64 device running Android 14+ with Termux from F-Droid.

**Verified** means both install paths tested end-to-end including authentication and basic operations. Test results: [tests/verification-results.txt](tests/verification-results.txt)

**Tested on your device?** [Submit a device report](https://github.com/ferrumclaudepilgrim/claude-code-android/issues/new?template=device_report.md) to fill in the gaps.

Claude Code is made by [Anthropic](https://www.anthropic.com). Official repo: [anthropics/claude-code](https://github.com/anthropics/claude-code).

---

## Skills for Android

This repo includes [Claude Code skills](https://code.claude.com/docs/en/skills) built specifically for Android/Termux environments.

| Skill | What It Does | How to Use |
|-------|-------------|-----------|
| `/doctor` | Diagnose your full Termux+Claude Code setup in one pass | Type `/doctor` in Claude Code |
| `/fix-ripgrep` | Fix broken Grep/Glob tools (missing arm64-android binary) | Type `/fix-ripgrep` in Claude Code |
| `termux-safe` | Auto-loaded constraints — prevents `sudo`, wrong paths, silent failures | Loads automatically |

### Installation

Copy the skills to your home directory so they work in any project:

```bash
git clone https://github.com/ferrumclaudepilgrim/claude-code-android.git
mkdir -p ~/.claude/skills
cp -r claude-code-android/.claude/skills/* ~/.claude/skills/
rm -rf claude-code-android    # Clean up — phone storage is finite
```

---

## The CLAUDE.md Template

Claude Code reads a CLAUDE.md file from your project root for persistent rules. The [template](CONSTITUTION-TEMPLATE.md) in this repo is ready to fork for Android/Termux — includes platform constraints, safety rules, and agent configuration.

---

## Contributing

Found a bug? Got it working on a new device? Know a better workaround?

- **Bug reports:** [Open an issue](https://github.com/ferrumclaudepilgrim/claude-code-android/issues/new?template=bug_report.md)
- **Device reports:** [Submit compatibility data](https://github.com/ferrumclaudepilgrim/claude-code-android/issues/new?template=device_report.md)
- **Improvements:** PRs welcome

---

## About This Project

This repo is built and maintained using Claude Code running on the same Android device it documents. The operator ([FerrumFluxFenice](https://github.com/FerrumFluxFenice)) directs the work — technical decisions and verification are human-directed. The commit history reflects that collaboration honestly.

## License

MIT. See [LICENSE](LICENSE).

---

<p align="center">
  <em>Built on a phone, in Termux, through proot, on ARM64, on Android.</em><br>
  <em>By a human and an AI, working together.</em>
</p>
