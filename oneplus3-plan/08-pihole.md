# 08 — Step 8: Pi-hole (Deferred)

Status: **Not yet designed. Lowest priority.**

- Only proceed if RAM headroom remains after the bot + Tailscale + SMB mount are running.
- Re-assess RAM/CPU headroom **on-device** first — the plan's 200–400MB combined estimate for the full workload (tailscaled, sshd, bot, optionally pihole-FTL + dnsmasq/unbound) is an **estimate, not measured**, against the OnePlus 3's 6GB pool.
- No design decisions made: install mechanism (native Termux vs proot-distro container), DNS port binding on unrooted Android, and upstream/DHCP questions are all open.
