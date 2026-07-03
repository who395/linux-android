# 06 — Step 6: Discord Bot as a Persistent Termux Service

Hard requirement: the bot runs **continuously** on wall power with no PC attached, surviving screen-off, Doze/app-kill behavior, and unattended device reboots.

---

## Mechanism (from architecture)

- `termux-wake-lock` + `nohup` (or `termux-services`) to keep the process alive while Termux is backgrounded.
- **Termux:Boot** to auto-start the bot after a device reboot — a wall-plug-only phone may reboot unattended (crash, update) and nothing else will restart the bot.
- Bot's local files live in Termux home (`~/`) or `~/storage/shared`; PC files via the rclone mount from [05-tailscale-smb.md](05-tailscale-smb.md).

## Background-persistence pattern (from README, adapted)

The README documents this pattern for keeping Home Assistant alive 24/7; the same pattern applies to the bot process:

```bash
# Option 1: Run in background with nohup
termux-wake-lock
nohup <your-bot-start-command> > ~/bot.log 2>&1 &

# Option 2: Auto-start on Termux launch (add to ~/.bashrc)
echo 'termux-wake-lock && nohup <your-bot-start-command> > ~/bot.log 2>&1 &' >> ~/.bashrc
```

From the README: "`termux-wake-lock` prevents Android from suspending the process. Plug your phone into a charger and it becomes a dedicated always-on server."

Note the `.bashrc` option only fires when Termux is *opened* — it does not survive a reboot on its own. That's the gap Termux:Boot fills (plan addition, not in the README).

---

## Threat model (user decision, 2026-07-03)

Vulnerability exposure is accepted as low: inbound is Tailscale-gated, outbound is only the Discord gateway, and nothing on the device renders untrusted web content. **The one untrusted input channel is Discord content into the bot** — user will lock this down.

Structural containment for the PC-write path (so a bot validation bug can't reach the rest of the PC):

- Share **one dedicated folder** on Windows, never a drive root.
- Use a Windows account for the SMB share that has rights to only that folder.
- Bot must resolve every target path and verify it stays under the mount root before writing — filenames/paths arriving via Discord (`../../`, absolute paths, odd Unicode) are filesystem input to the PC.

With that containment, the ROM's 2024-era security string is genuinely irrelevant to this workload — exploiting stale patches would require an attacker already on the tailnet.

---

## Not yet verified (do not assume)

- Termux:Boot reliability specifically on the MSe1969 ROM build — not checked.
- Whether Termux:Boot survives this particular ROM's first-boot/update cycle reliably — not checked. The ROM has OTA support with monthly builds (confirmed from the XDA thread capture), so this recurs monthly as unattended reboots — treat the first OTA as a deliberate test.
- Whether Doze/battery-optimization exemption settings are still needed even on a custom ROM (likely yes, just less aggressive than stock) — not checked.
