# 01 — Decided Architecture

## Key architectural correction

**Original ask implied:** USB cable connection gives the bot access to PC files.

**Why that doesn't work:** MTP (the protocol phones use over USB-to-PC) is one-directional by design — it lets the **PC** browse the **phone's** storage. It does not let the **phone** browse the **PC's** filesystem. There is no "reverse MTP." A background Termux process has no API into an MTP session to read a Windows folder.

**Resolution:** PC-file access needs a network mechanism (SMB share + Tailscale), not the cable. Once that exists, it works whether or not the phone is physically plugged in — which is also strictly more useful (works over WiFi from anywhere on the tailnet, not just at the desk). USB stays reserved for manual bulk transfer via plain MTP in Windows Explorer.

---

## Architecture table

| Need | Mechanism | Notes |
|---|---|---|
| Discord bot runs continuously | Termux background service: `termux-wake-lock` + `nohup` (or `termux-services`) + **Termux:Boot** app for auto-start on device reboot | Termux:Boot is a separate companion app from the same F-Droid maintainer. Needed because a wall-plug-only phone may reboot unattended (crash, update) and nothing else will restart the bot. |
| Bot reads/writes local phone files | Termux home (`~/`) or `~/storage/shared` after `termux-setup-storage` | No networking involved. |
| Bot reads/writes PC files | Windows 11 SMB share (Properties → Sharing) + Tailscale on PC, mounted into Termux via **`rclone mount`** (FUSE-based) pointed at PC's Tailscale IP/MagicDNS name | `rclone` chosen over `mount -t cifs` because Termux is unrooted and a real CIFS mount needs root/kernel module access; `rclone`'s FUSE approach avoids that. **Confirmed: always-on via Tailscale, not cable-gated.** |
| Manual bulk file transfer | Plain MTP, phone cabled to PC, drag-and-drop in Windows Explorer | Separate from the bot's automated PC access — intentionally two different mechanisms for two different jobs (human-driven vs bot-driven). |
| iPhone edits phone files | Tailscale app on iPhone (official, iOS-supported) + SFTP client that registers as an iOS Files provider — **Secure ShellFish** suggested (common choice for this) — connecting to the phone's `sshd` (port 8022, already part of base Termux setup) | Works over Tailscale, so it works on any network, not just same-WiFi. |
| Pi-hole (if headroom remains) | Not yet designed | Lowest priority; revisit after bot + Tailscale + SMB footprint is measured on-device. See [08-pihole.md](08-pihole.md). |

---

## What "MTP" is (background, for reference)

Media Transfer Protocol — the default USB-cable protocol modern Android phones use instead of exposing a raw mountable filesystem. The phone stays in control of storage and mediates access through a protocol layer rather than handing over a raw block device; Windows sees it via the built-in WPD driver stack (no extra install needed), appearing as a virtual folder, not a drive letter. It's slower and less standard than a real filesystem, and it does not support the phone reading the PC's files — protocol is PC-reads-phone only.

## USB scenario — resolved

User confirmed the USB use case is "plug into PC and get file transfer" = plain **MTP**, not OTG host-mode (phone acting as USB host for an external drive). This means the earlier-flagged "USB host mode kernel support" concern **does not apply** — MTP is device-mode, standard, and expected to work on stock or any custom ROM without special kernel support. No verification needed on this point.
