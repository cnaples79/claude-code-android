# Claude Code on Android

<p align="center">
  <img src="logo.jpg" alt="Claude Code on Android" width="200">
</p>

<p align="center">
  <strong>Run Claude Code natively on Android — no root, no emulator, no cloud VM.</strong>
</p>

<p align="center">
  <a href="INSTALL.md">Install Guide</a> · <a href="TROUBLESHOOTING.md">Troubleshooting</a> · <a href="CONSTITUTION-TEMPLATE.md">CLAUDE.md Template</a>
</p>

---

## Quick Start

Four commands. Termux open. Go.

```bash
pkg install nodejs git proot -y
export TMPDIR=$PREFIX/tmp
npm install -g @anthropic-ai/claude-code
proot -b $PREFIX/tmp:/tmp claude
```

That's it. Claude Code is running on your phone.

> **Note:** The Quick Start commands work for this session. Add TMPDIR to your .bashrc (shown below) to make it permanent.

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

### 1. proot-distro is broken on Android 16 (Android 16 specific)

The kernel's security model breaks stdout file descriptor binding inside guest distributions. Processes launch but produce no output. **There is no fix inside the guest distro.** The solution: skip guest distros entirely. Run Claude Code natively in Termux.

### 2. /tmp doesn't exist

Claude Code hardcodes `/tmp` for sockets and IPC. On Android, `/tmp` isn't writable. Without it, Claude Code fails silently — no error, no crash log, just nothing. The fix: `proot -b $PREFIX/tmp:/tmp` remaps the path at the syscall level. No root required.

### 3. Node.js v24 hangs on ARM64

Node.js v24 hangs on startup under Termux on ARM64. The cause is unclear but upgrading to v25+ fixes it. Termux ships v25 by default now, so fresh installs avoid this.

---

## What's In This Repo

| File | What It Is |
|------|-----------|
| **[INSTALL.md](INSTALL.md)** | Complete step-by-step install guide with verification |
| **[TROUBLESHOOTING.md](TROUBLESHOOTING.md)** | Common failures with symptoms, causes, and fixes |
| **[CONSTITUTION-TEMPLATE.md](CONSTITUTION-TEMPLATE.md)** | A CLAUDE.md template for giving Claude Code persistent rules and identity on Android |
| **[LICENSE](LICENSE)** | MIT |

---

## Known Constraints

Running on a phone means real limits. Know them upfront:

| Constraint | Impact | Workaround |
|-----------|--------|-----------|
| No root | No `sudo`, no ports below 1024 | Use ports 1024+, skip anything that needs root |
| No systemd | No services, no daemons the normal way | Use `crond` or shell scripts for persistence |
| ~512MB Node.js heap | Large datasets must stream | Don't buffer — stream and process incrementally |
| ~1024 file descriptors | Heavy I/O can hit EMFILE | Limit concurrent processes |
| Phantom process killer | Android kills excess background processes | Limit to 2-3 background processes max |
| /tmp is volatile | proot crash = mount gone | Never store persistent state in /tmp |

See [TROUBLESHOOTING.md](TROUBLESHOOTING.md) for detailed fixes.

---

## Device Compatibility

Verified working on:

| Device | Android | Kernel | Termux Source | Node.js | Status |
|--------|---------|--------|--------------|---------|--------|
| aarch64 Android device | 14+ | 6.12.x | F-Droid | v25.8.1 | Works |

**Tested on your device?** [Submit a device report](../../issues/new?template=device_report.md) so others know.

Claude Code is made by [Anthropic](https://www.anthropic.com). Official repo: [anthropics/claude-code](https://github.com/anthropics/claude-code).

---

## The CLAUDE.md Template

Claude Code reads a CLAUDE.md file from your project root for persistent rules. The [template](CONSTITUTION-TEMPLATE.md) in this repo is ready to fork for Android/Termux — includes platform constraints, safety rules, and agent configuration.

---

## Contributing

Found a bug? Got it working on a new device? Know a better workaround?

- **Bug reports:** [Open an issue](../../issues/new?template=bug_report.md)
- **Device reports:** [Submit compatibility data](../../issues/new?template=device_report.md)
- **Improvements:** PRs welcome

---

## License

MIT. See [LICENSE](LICENSE).

---

<p align="center">
  <em>Built on a phone, in Termux, through proot, on ARM64, on Android.</em><br>
  <em>Because the only computer you need is the one in your pocket.</em>
</p>
