# Installing Claude Code on Android

A complete, reproducible guide to running Claude Code on an aarch64 Android device using Termux. Every command has been tested on real hardware. Follow these steps in order and it will work.

---

## Prerequisites

Before you begin, confirm you have the following:

| Component | Requirement |
|-----------|-------------|
| **Device** | aarch64 Android device (ARM64) |
| **OS** | Android 16 |
| **Kernel** | 6.12.x (`uname -r` to verify) |
| **Terminal** | [Termux from F-Droid](https://f-droid.org/en/packages/com.termux/) — **not** the Play Store version, which is outdated and will fail |
| **Subscription** | Claude Max or Claude Pro (provides the API access Claude Code requires) |
| **Network** | Active internet connection (Claude Code streams from Anthropic's API) |

> **Warning:** The Play Store version of Termux has not been updated since 2020 and does not support current package repositories. You must use F-Droid or install the `.apk` directly from the [Termux GitHub releases](https://github.com/termux/termux-app/releases).

---

## Why This Is Hard

Running Claude Code on Android 16 requires solving three distinct problems that have stopped others. Understanding them will save you hours.

### Problem 1: Android 16 Breaks proot-distro

The obvious approach — install a full Linux distribution inside Termux using `proot-distro` — fails on Android 16. The kernel's updated security model breaks proot's stdout file descriptor binding inside guest distributions. Processes launch but produce no output, or hang indefinitely. This is a kernel-level restriction, not a configuration issue. There is no fix within the guest distro. On this kernel, `proot-distro` is a dead end for interactive CLI tools.

### Problem 2: The /tmp Restriction

Claude Code hardcodes `/tmp` for socket files, IPC, and ephemeral state. On Android, `/tmp` either doesn't exist or isn't writable from within Termux's sandbox. Without it, Claude Code fails silently — no error message, no crash log. It simply doesn't start, or starts and immediately loses the ability to communicate with its own subprocesses.

### Problem 3: The Node.js v24 Hang

Multiple users have reported that Claude Code hangs on startup with Node.js v24 on ARM64 under Termux. The exact cause is unclear — likely related to how Node's event loop interacts with Android's process model — but upgrading to Node.js v25+ resolves it. Termux's current `pkg` repository ships v25, so a fresh install avoids this entirely. If you have an older Node version pinned, upgrade it.

### Why This Combination Works

The solution skips the guest distro entirely. Install Claude Code natively inside Termux, where Node.js runs without the proot-distro stdout breakage. Handle the `/tmp` problem at launch time only, using a minimal `proot` bind mount — not a full guest OS, just a single path remap. This is lighter, faster, and immune to the guest-distro kernel restrictions.

---

## Environment Reference

Key paths and versions for a working installation:

- **Architecture:** aarch64 (ARM64)
- **Kernel:** 6.12.x (Android 16) — verify with `uname -r`
- **Shell:** Termux
- **Home:** `/data/data/com.termux/files/home`
- **Prefix:** `/data/data/com.termux/files/usr`
- **Node.js:** v25+
- **proot:** 5.1.107+

There is no root access. There is no systemd. There is no `/tmp` in the way most Unix programs expect. The filesystem paths are deeply nested inside Android's app sandbox.

---

## Step 1: Install Dependencies

Open Termux and run:

```bash
pkg install nodejs git curl -y
```

This installs Node.js v25+, git, and curl. All three are required — Node.js runs Claude Code, git is needed for repository operations, and curl is used during authentication flows.

---

## Step 2: Set TMPDIR

```bash
export TMPDIR=$PREFIX/tmp
```

**This is critical.** Termux does not set `TMPDIR` by default. Without it, npm has no writable temporary directory. The install will either fail silently, produce a corrupted installation, or appear to succeed while leaving Claude Code unable to start. This single missing environment variable is the most common point of failure in Termux Node.js setups.

`$PREFIX` resolves to `/data/data/com.termux/files/usr`. The `tmp` directory inside it is writable by Termux processes.

---

## Step 3: Install Claude Code

```bash
npm install -g @anthropic-ai/claude-code
```

This installs Claude Code globally. With `TMPDIR` set correctly, npm can stage files, compile any native addons, and complete the installation cleanly.

---

## Step 4: Launch with proot

Claude Code hardcodes `/tmp` for runtime state. The fix is `proot` — a userspace path remapper that requires no root privileges. It intercepts system calls and makes `/tmp` point to Termux's writable tmp directory.

Install proot if you don't already have it:

```bash
pkg install proot -y
```

Launch Claude Code:

```bash
proot -b $PREFIX/tmp:/tmp claude
```

This single invocation binds Termux's writable tmp directory to `/tmp`, allowing Claude Code to operate as if it were on a standard Linux system. No root. No containers. No virtualization. Just syscall interception.

On first launch, Claude Code will prompt you to authenticate with your Anthropic account.

---

## Step 5: Create the Alias

Add this to your `~/.bashrc` so you never have to remember the proot invocation:

```bash
echo "alias claude-android='proot -b \$PREFIX/tmp:/tmp claude'" >> ~/.bashrc
source ~/.bashrc
```

Then launch with:

```bash
claude-android
```

---

## Verification

After completing the steps above, run these commands to confirm your setup matches the verified configuration:

```bash
# Node.js — must be v25+
node -v

# npm — should be v11+
npm -v

# Claude Code — confirms the binary is installed and on PATH
claude --version

# proot — confirms proot is available
proot --help 2>&1 | head -1

# TMPDIR — must be set to a writable path
echo $TMPDIR

# /tmp bind — confirms proot remapping works
proot -b $PREFIX/tmp:/tmp ls /tmp
```

Expected output pattern:

```
v25.x.x
11.x.x
2.x.x (Claude Code)
Usage: proot [...]
/data/data/com.termux/files/usr/tmp
[contents of tmp directory]
```

If `node -v` shows v24 or below, upgrade with `pkg upgrade nodejs`. If `echo $TMPDIR` is empty, add `export TMPDIR=$PREFIX/tmp` to your `~/.bashrc` and source it.

---

## Tested On

Community-reported working configurations:

| Device | Android Version | Kernel | Termux Source | Node.js | Status |
|--------|----------------|--------|---------------|---------|--------|
| aarch64 Android device | Android 16 | 6.12.x | F-Droid | v25.8.1 | Verified |

If you have tested this guide on your device, please open an issue or PR to add your configuration to this table.

---

*Last verified: March 18, 2026*
