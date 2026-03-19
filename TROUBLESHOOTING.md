# Troubleshooting Claude Code on Android

This guide covers problems specific to running Claude Code on **Android 16 + Termux** (aarch64/ARM64). Each entry describes a symptom, its cause, and how to fix it.

If you haven't installed yet, see [INSTALL.md](INSTALL.md) first.

---

### Claude Code hangs on startup

**Problem:** You run `claude` and it hangs indefinitely — no prompt, no output, no error.

**Cause:** `TMPDIR` is not set. Claude Code and Node.js need a writable temporary directory. Termux does not set one by default, so npm's internal operations and Claude Code's IPC sockets have nowhere to go.

**Fix:**

```bash
export TMPDIR=$PREFIX/tmp
```

Add it to `~/.bashrc` so it persists:

```bash
echo 'export TMPDIR=$PREFIX/tmp' >> ~/.bashrc
source ~/.bashrc
```

---

### Claude Code won't start, no error

**Problem:** You run `claude` and nothing happens. No error message, no crash log. The command returns silently or the process exits immediately.

**Cause:** `/tmp` is not writable. Claude Code hardcodes `/tmp` for socket files, IPC, and ephemeral state. On Android, `/tmp` either doesn't exist or isn't writable from Termux's sandbox.

**Fix:** Launch Claude Code through `proot` with a bind mount:

```bash
pkg install proot -y
proot -b $PREFIX/tmp:/tmp claude
```

Create an alias so you don't have to type it every time:

```bash
echo "alias claude-proot='proot -b \$PREFIX/tmp:/tmp claude'" >> ~/.bashrc
source ~/.bashrc
```

---

### proot-distro doesn't work

**Problem:** You installed a Linux distribution via `proot-distro` (Ubuntu, Debian, etc.) and Claude Code inside the guest produces no output, hangs, or behaves erratically.

**Cause:** Android 16's kernel (6.12.x) breaks proot's stdout file descriptor binding inside guest distributions. This is a kernel-level restriction, not a configuration issue. There is no fix within the guest distro.

**Fix:** Do not use `proot-distro` for Claude Code. Install and run everything natively in Termux:

```bash
pkg install nodejs git curl proot -y
export TMPDIR=$PREFIX/tmp
npm install -g @anthropic-ai/claude-code
proot -b $PREFIX/tmp:/tmp claude
```

You only need `proot` for the single bind mount (`/tmp`), not a full guest OS.

---

### Node.js v24 hangs

**Problem:** Claude Code hangs on startup with Node.js v24 installed. The process appears to start but never becomes interactive.

**Cause:** Node.js v24 has an event loop issue on ARM64 under Termux. The exact mechanism is unclear, but it affects how Node's event loop interacts with Android's process model.

**Fix:** Upgrade to Node.js v25 or later:

```bash
pkg upgrade nodejs -y
node -v  # should show v25.x.x or higher
```

If `pkg upgrade` doesn't move you to v25, check that your Termux package repositories are current. The F-Droid version of Termux ships v25+ in its default repo.

---

### Process killed randomly

**Problem:** Claude Code (or its subprocesses) dies unexpectedly mid-session. No error, no crash — the process just disappears.

**Cause:** Android's phantom process killer. Android limits background processes to approximately 32 across all apps. When Termux spawns multiple Node.js processes (Claude Code, subagents, language servers), the OS silently kills excess processes.

**Fix:**

1. Minimize background apps while using Claude Code.
2. Limit concurrent subagents and child processes.
3. Enable the developer option to disable the restriction:

   **Settings → Developer Options → Disable child process restrictions** (toggle on)

   If you don't see Developer Options, go to **Settings → About phone** and tap **Build number** seven times.

---

### EMFILE errors

**Problem:** You see `EMFILE: too many open files` errors during operation.

**Cause:** The file descriptor limit under proot is approximately 1024. Heavy I/O, many open sockets, or spawning lots of processes can exhaust this limit.

**Fix:**

- Avoid spawning unnecessary background processes.
- Close unused terminal sessions.
- If running multiple tools simultaneously, reduce parallelism.
- Restart Claude Code to release leaked file descriptors.

There is no way to raise the FD limit under proot on Android without root access.

---

### npm install fails silently

**Problem:** `npm install -g @anthropic-ai/claude-code` appears to complete but Claude Code isn't installed, or the install produces no output and no binary.

**Cause:** `TMPDIR` is not set. Without a writable temporary directory, npm cannot stage files or compile native addons. It fails silently rather than reporting an error.

**Fix:**

```bash
export TMPDIR=$PREFIX/tmp
npm install -g @anthropic-ai/claude-code
```

Always set `TMPDIR` before any npm operation in Termux. Add it to `~/.bashrc` to make it permanent.

---

### /tmp data lost

**Problem:** Files you wrote to `/tmp` are gone. In-progress work that relied on `/tmp` state is lost.

**Cause:** The proot bind mount is not a real filesystem mount — it's syscall interception. If proot crashes or the session ends, the mapping disappears. `/tmp` under proot is ephemeral.

**Fix:** Never store important state in `/tmp`. Treat it as disposable. Write anything you need to keep into your project directory or another persistent path under `$HOME`.

---

### Play Store Termux doesn't work

**Problem:** You installed Termux from the Google Play Store and packages fail to install, repositories are missing, or the app behaves unexpectedly.

**Cause:** The Play Store version of Termux has not been updated since 2020. It does not support current package repositories, and its bundled tools are too old to run Claude Code.

**Fix:** Uninstall the Play Store version and install Termux from one of these sources:

- [F-Droid](https://f-droid.org/en/packages/com.termux/)
- [Termux GitHub releases](https://github.com/termux/termux-app/releases) (direct APK)

After installing, run:

```bash
pkg update && pkg upgrade -y
```
