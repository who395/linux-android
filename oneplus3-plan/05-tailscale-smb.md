# 05 — Steps 4–5: Tailscale + Windows SMB Share + rclone Mount

Purpose: give the bot always-on read/write access to selected PC folders, independent of the USB cable. (See [01-architecture.md](01-architecture.md) for why the cable can't do this — MTP is PC-reads-phone only.)

**Confirmed requirement:** always available via Tailscale, not gated to only-when-cabled.

---

## Step 4 — Install and configure Tailscale on phone + PC

- Tailscale on the **Windows 11 PC** (standard installer).
- Tailscale on the **phone**: runs as a standalone Android APK using the standard AOSP `VpnService` framework API — it does **not** run inside Termux and requires no special integration with this ROM. Termux processes (bot, sshd) simply route over whatever path Tailscale establishes at the OS level. **(Architectural reasoning, not device-specific verification.)**
- Result: PC reachable from the phone by Tailscale IP or MagicDNS name from any network.

## Step 5 — Windows SMB share + `rclone mount` from Termux

1. On Windows 11: share the target folder(s) — folder **Properties → Sharing**.
2. In Termux: install `rclone`, configure an SMB remote pointing at the PC's Tailscale IP/MagicDNS name.
3. Mount it with **`rclone mount`** (FUSE-based).

**Why `rclone` and not `mount -t cifs`:** Termux is unrooted, and a real CIFS mount needs root/kernel module access; rclone's FUSE approach avoids that.

---

## Reliability findings: Termux + Tailscale

Search query used: `Termux Tailscale LineageOS 18.1 oneplus3 issues battery background` — results are **snippets, not fetched pages**, except where noted.

- **Real, current, sourced concern (Tailscale's own GitHub issue tracker, snippet-level):** multiple **open, unresolved** Android battery-drain issues exist — notably #17547 (Oct 2025, drain when app can't reach the internet — retry-loop pattern) and #13725 (Oct 2024, drain triggered specifically by split-tunneling). None are OnePlus3/msm8996/this-ROM-specific; they span various current Android devices (Pixel 5/7 Pro, OnePlus 9 Pro).
- **Why lower-impact here:** these are battery-drain reports; the device is wall-powered, so battery impact is moot. The underlying cause (wake-lock/CPU churn from retries) could theoretically cause heat/occasional instability on old hardware, but **this is speculative — not confirmed for this device.**
- **If issues appear post-setup:** Tailscale's official mobile troubleshooting doc exists at `https://tailscale.com/docs/reference/troubleshooting/mobile` (title fetched, body not read) and covers this exact issue class.
- **Remaining unverified:** whether `VpnService`-based Tailscale interacts correctly with this specific kernel/ROM combo in practice — no direct reports found either way, and absence of complaints isn't confirmation.

---

## Not yet verified (from the plan's unverified list — do not assume)

- FUSE support for `rclone mount` on the chosen ROM/kernel — expected to work since FUSE is standard on modern Android, but this is an expectation, not a confirmation.
- Tailscale client's own CVE/security history — not checked; flagged as a gap given Tailscale is now the primary trust boundary for the whole setup.
