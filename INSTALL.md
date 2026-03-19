# Installing Claude Code on Android

A complete, reproducible guide to running Claude Code on an aarch64 Android device using Termux. Every command has been tested on real hardware. Follow these steps in order and it will work.

---

## Prerequisites

Before you begin, confirm you have the following:

| Component | Requirement |
|-----------|-------------|
| **Device** | aarch64 Android device (ARM64) |
| **OS** | Android 14+ |
| **Kernel** | 6.12.x (`uname -r` to verify) |
| **Terminal** | [Termux from F-Droid](https://f-droid.org/en/packages/com.termux/) — **not** the Play Store version, which is outdated and will fail |
| **Subscription** | Claude Max or Claude Pro (provides the API access Claude Code requires) |
| **Network** | Active internet connection (Claude Code streams from Anthropic's API) |

> **Warning:** The Play Store version of Termux has not been updated since 2020 and does not support current package repositories. You must use F-Droid or install the `.apk` directly from the [Termux GitHub releases](https://github.com/termux/termux-app/releases).

---

## Why This Is Hard

Android requires solving problems that have stopped others. Some are universal, some are Android 16-specific. Understanding them will save you hours.

> **Note:** Problems 1 and 3 are Android 16-specific. The /tmp fix (Problem 2) applies to all Android versions.

### Problem 1: proot-distro Is a Detour, Not a Dead End

The obvious approach — install a full Linux distribution inside Termux using `proot-distro` — works but is unnecessary overhead. A TCGETS2 ioctl bug that broke proot-distro on kernel 6.12 was fixed in proot 5.1.107-66 (October 2025). Guest distros install and run correctly with current proot versions.

However, a full guest OS is not required for Claude Code. Claude Code only needs a writable `/tmp` — which a single proot bind mount provides without the overhead of an entire Linux rootfs. The native Termux approach is lighter, faster, and avoids the storage cost of maintaining a guest distribution.

> **Note:** You will see `proot warning: can't sanitize binding "/proc/self/fd/1"` during proot-distro operations. This is harmless — proot cannot resolve the `/proc/self/fd` symlink inside the guest, but stdout functions correctly regardless.
>
> If you prefer a full Linux environment, see [Path B: proot-distro Ubuntu](#path-b-proot-distro-ubuntu) at the end of this guide.

### Problem 2: The /tmp Restriction

Claude Code hardcodes `/tmp` for socket files, IPC, and ephemeral state. On Android, `/tmp` either doesn't exist or isn't writable from within Termux's sandbox. Without it, Claude Code fails silently — no error message, no crash log. It simply doesn't start, or starts and immediately loses the ability to communicate with its own subprocesses.

### Problem 3: The Node.js v24 Hang

Multiple users have reported that Claude Code hangs on startup with Node.js v24 on ARM64 under Termux. The exact cause is unclear — likely related to how Node's event loop interacts with Android's process model — but upgrading to Node.js v25+ resolves it. Termux's current `pkg` repository ships v25, so a fresh install avoids this entirely. If you have an older Node version pinned, upgrade it.

### Why This Combination Works

The solution skips the guest distro — not because it's broken (it isn't, as of proot 5.1.107-66), but because it's unnecessary overhead. Install Claude Code natively inside Termux, where Node.js runs directly on the host. Handle the `/tmp` problem at launch time only, using a minimal `proot` bind mount — not a full guest OS, just a single path remap. This is lighter, faster, and avoids the storage and complexity cost of maintaining a guest distribution.

---

## Environment Reference

Key paths and versions for a working installation:

- **Architecture:** aarch64 (ARM64)
- **Kernel:** 6.12.x (Android 16) — earlier Android versions use different kernels — verify with `uname -r`
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
pkg install nodejs git curl proot -y
```

This installs Node.js v25+, git, curl, and proot. All four are required — Node.js runs Claude Code, git is needed for repository operations, curl is used during authentication flows, and proot handles the `/tmp` bind mount at launch.

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

This installs Claude Code globally via npm. With `TMPDIR` set correctly, npm can stage files and complete the installation cleanly.

> **Note:** Anthropic now offers a native installer (`curl -fsSL https://claude.ai/install.sh | bash`) as the preferred installation method. However, the native installer targets standard Linux platforms and may not detect Termux correctly. The npm method remains reliable for native Termux installs. If you use [Path B (proot-distro Ubuntu)](#path-b-proot-distro-ubuntu), the native installer works there since the guest reports as standard Linux.

---

## Step 4: Launch with proot

Claude Code hardcodes `/tmp` for runtime state. The fix is `proot` — a userspace path remapper that requires no root privileges. It intercepts system calls and makes `/tmp` point to Termux's writable tmp directory.

proot was installed in Step 1.

Launch Claude Code:

```bash
proot -b $PREFIX/tmp:/tmp claude
```

This single invocation binds Termux's writable tmp directory to `/tmp`, allowing Claude Code to operate as if it were on a standard Linux system. No root. No containers. No virtualization. Just syscall interception.

On first launch, Claude Code will prompt you to authenticate. A URL will appear in your terminal — open it in your phone's browser to complete OAuth. If authentication fails, see the [OAuth troubleshooting entry](TROUBLESHOOTING.md#oauth--authentication-fails-on-first-launch).

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

Tested on Android 16. Expected to work on Android 14+ but not yet verified on earlier versions. If you have tested this guide on your device, please open an issue or PR to add your configuration to this table.

---

---

## Path B: proot-distro Ubuntu

An alternative approach: install a full Ubuntu guest inside Termux, then install Claude Code inside it. This avoids the `/tmp` bind mount and ripgrep symlink entirely, at the cost of more disk space and setup time.

### When to use Path B

- You want a full Linux environment (apt, standard paths, /tmp works natively)
- You plan to run other Linux tools alongside Claude Code
- You prefer the native installer over npm

### Setup

```bash
# In Termux (not inside Claude Code)
pkg install proot-distro -y
proot-distro install ubuntu
proot-distro login ubuntu
```

Inside the Ubuntu guest:

```bash
# Install Claude Code via the native installer
curl -fsSL https://claude.ai/install.sh | bash

# Launch Claude Code
claude
```

That's it. No `/tmp` workaround needed — Ubuntu has a native `/tmp`. No ripgrep symlink needed — the Termux system ripgrep is accessible via PATH.

### Path B trade-offs

| | Path A (Native Termux) | Path B (proot-distro Ubuntu) |
|---|---|---|
| Setup time | ~2 minutes | ~10-15 minutes |
| Disk usage | Minimal | ~500MB+ for Ubuntu rootfs |
| /tmp workaround | Required (proot bind mount) | Not needed |
| ripgrep fix | Required (symlink) | Not needed |
| Install method | npm | Native installer (curl) |
| Auth flow | Browser auto-opens | Manual URL copy/paste |
| Overhead | Minimal | Full guest OS layer |
| Best for | Quick setup, light usage | Full Linux environment |

### Path B notes

- **Authentication:** The browser will not auto-open from inside the guest. Copy the auth URL from the terminal and paste it into your phone's browser manually.
- **The warning `can't sanitize binding "/proc/self/fd/1"`** appears during login. It is harmless — stdout works correctly.
- **Samsung One UI 8 users:** There is a known performance regression when Termux is backgrounded (proot-distro issue [#567](https://github.com/termux/proot-distro/issues/567)). Keep Termux in the foreground or split-screen for best performance.
- **You cannot run proot-distro from inside Claude Code** if Claude Code was launched with `proot -b`. proot-distro detects nesting and refuses. Run proot-distro commands from a separate Termux session.

### Verified Path B configuration

| Component | Version |
|-----------|---------|
| proot-distro | 4.38.0 |
| Guest OS | Ubuntu 25.10 (Questing Quokka) |
| Claude Code | 2.1.79 (native installer) |
| Kernel | 6.12.30 (Android 16) |

---

*Last verified: March 19, 2026*
