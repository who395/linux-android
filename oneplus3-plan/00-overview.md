# 00 — Overview: OnePlus 3 → Discord Bot / File Server

Last updated: 2026-07-03
Status: **ROM and build decided; flashing not yet executed. Battery health check available as a pre-flash step.**

---

## Goal (as stated by user)

Repurpose a OnePlus 3 into an always-on device that:

- Runs a Discord bot **continuously**, including on wall power with no PC attached.
- Bot reads/writes **local files** on the phone.
- Bot also reads/writes **some files on the user's Windows 11 PC**.
- Supports **manual file transfer** by plugging the phone into the PC via USB.
- Is editable **from an iPhone** (remote file access/editing).
- _If_ RAM headroom allows after the above: run Pi-hole too.
- Explicitly **not** running a Home Assistant dashboard.

---

## Standing assumptions

- PC is **Windows 11** (user's stated default environment).
- "Work without the PC" means the bot must survive both:
  - physical USB disconnection, and
  - Android backgrounding/Doze/app-kill behavior — not just the cable being unplugged.
- PC-file access from the bot should be **always available via Tailscale**, not gated to only-when-cabled. **(Confirmed by user.)**
- The architecture (see [01-architecture.md](01-architecture.md)) is architectural reasoning from how MTP/Termux/Tailscale/SMB work, not sourced content — flagged inline where relevant.

---

## Step order / file index

| Step | File                                                               | What                                                                    |
| ---- | ------------------------------------------------------------------ | ----------------------------------------------------------------------- |
| —    | [01-architecture.md](01-architecture.md)                           | Decided architecture + the MTP correction                               |
| 1    | [02-battery-health-check.md](02-battery-health-check.md)           | Optional, zero-risk pre-flash battery health check + mitigation options |
| 2    | [03-rom-decision-and-flash.md](03-rom-decision-and-flash.md)       | ROM decision (**decided**) + Windows 11 flashing procedure              |
| 3    | [04-termux-setup.md](04-termux-setup.md)                           | Termux, Termux:Boot, Termux:X11 install (README-derived)                |
| 4–5  | [05-tailscale-smb.md](05-tailscale-smb.md)                         | Tailscale on phone + PC, Windows SMB share, `rclone mount`              |
| 6    | [06-discord-bot-service.md](06-discord-bot-service.md)             | Bot as persistent Termux service (README-derived persistence patterns)  |
| 7    | [07-ssh-iphone-access.md](07-ssh-iphone-access.md)                 | sshd + iPhone editing over Tailscale (README-derived SSH details)       |
| 8    | [08-pihole.md](08-pihole.md)                                       | Pi-hole — deferred until headroom measured                              |
| —    | [09-open-questions-unverified.md](09-open-questions-unverified.md) | Everything not yet verified — do not assume these work                  |

---

## Next steps

1. **Battery health check (optional, zero-risk)** — run now, on current stock OxygenOS, before flashing. See [02-battery-health-check.md](02-battery-health-check.md).
2. Execute the install/flash steps in [03-rom-decision-and-flash.md](03-rom-decision-and-flash.md).
3. Set up Termux, Termux:Boot, Termux:X11 per the original setup doc.
4. Install and configure Tailscale on phone + PC.
5. Set up Windows SMB share + `rclone mount` from Termux.
6. Set up Discord bot as a persistent Termux service.
7. Set up sshd + confirm Secure ShellFish (or equivalent) from iPhone over Tailscale.
8. Re-assess RAM/CPU headroom on-device before deciding on Pi-hole.
