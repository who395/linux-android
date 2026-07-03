# 04 — Step 3: Termux, Termux:Boot, Termux:X11 Setup

Source: copied from the repo [README.md](../README.md) (the "original setup doc" the plan references), trimmed to what this project needs. Desktop-environment/GPU/Wine content is omitted — the bot/server use case doesn't need a GUI, though the script installs one; Termux:X11 is listed because the plan names it.

---

## Hardware requirements (from README)

- Android phone with an **arm64 (64-bit)** processor
- **3 GB+ RAM** recommended — the OnePlus 3 has 6 GB, comfortably above this
- **5–10 GB** of free storage (more if you install Wine — not needed here)

## Required apps (from README)

| App | Where to Get It |
|---|---|
| **Termux** | [F-Droid](https://f-droid.org/en/packages/com.termux/) — **do not use the Play Store version, it is outdated** |
| **Termux-X11** | [GitHub Releases](https://github.com/termux/termux-x11/releases) — download the latest `.apk` |
| **Termux:Boot** | F-Droid, same maintainer as Termux (plan requirement — auto-restarts services after unattended reboot; not covered by the README) |

Grant all apps any permissions they request.

## Step A — Pre-upgrade Termux (important — do this first)

Open Termux and run:

```bash
termux-wake-lock
pkg upgrade -y
```

The `termux-wake-lock` command keeps Termux alive when your screen turns off — without it, Android can kill the process mid-install. The `pkg upgrade` brings your base system up to date before the script runs, preventing a known crash involving `libpcre` and `libandroid-selinux`.

## Step B — Download and run the setup script

```bash
curl -O https://raw.githubusercontent.com/mayukh4/linux-anroid/main/termux-linux-setup.sh
chmod +x termux-linux-setup.sh
bash termux-linux-setup.sh
```

The script asks you to choose a desktop environment and whether you want Wine. For this project: pick a light DE (XFCE4 or LXQt) since the desktop is incidental, and **skip Wine**. Installation takes 10–30 minutes; a full log is saved to `~/termux-setup.log`.

Relevant items the script installs (from README's "What Gets Installed"):

| Component | Why it matters for this plan |
|---|---|
| **OpenSSH** | SSH server — needed for iPhone access (see [07-ssh-iphone-access.md](07-ssh-iphone-access.md)) |
| **Python 3 + pip** | Likely runtime for the Discord bot |
| **Git, wget, curl** | Standard tooling |

## Step C — Phone storage access for the bot

```bash
termux-setup-storage
```

Grants Termux access to `~/storage/shared` — the bot's local-file read/write location (see architecture table in [01-architecture.md](01-architecture.md)).

## Troubleshooting (from README, relevant subset)

- **Script exits mid-install without a clear error** — check `~/termux-setup.log`; the last line names the failing package.
- **"library not found" or "cannot link executable" during install** — the libpcre crash. Close Termux completely, reopen, run `pkg upgrade -y`, re-run the script.
- **"unmet dependencies" / "Conflicts"** — the script's `safe_install_pkg` reads apt conflict declarations and skips packages that would break the system; if it still fails, check the log.
