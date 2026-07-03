# 09 — Open Questions & Explicitly Unverified Items

**Do not assume these work.** Consolidated from the plan; item-specific copies also appear in the relevant step files.

## Triage — what actually blocks starting

Verdict: **nothing blocks starting anymore.** The one pre-start item — reading the [XDA thread](https://xdaforums.com/t/rom-unofficial-11-0-microg-signed-hardened-lineageos-18-1-oneplus-3-3t.4347693/) including its spoiler sections — was completed 2026-07-03; install steps are verified and the firmware prerequisite is resolved (fully update stock OxygenOS before unlocking).
- **Only testable by proceeding** (starting is the fastest way to resolve them): zram (`cat /proc/swaps` post-flash), FUSE/`rclone mount` (30-second test once Termux is up), Termux:Boot reliability (on-device reboot test), Tailscale `VpnService` on this ROM (no reports exist either way).
- **Post-setup tweaks, not blockers:** Doze/battery-optimization exemptions (settings toggle + overnight soak test), Tailscale CVE review (doesn't change the architecture).
- **Only if pursuing battery options:** battery sysfs root requirement (just attempt it), battery-bypass PMIC support (irrelevant unless Option A is chosen, which isn't recommended).

## Resolved since last revision

- **ROM choice — DECIDED**: MSe1969 LineageOS 18.1 microG build, file `lineage-18.1-20260414-UNOFFICIAL-microG-signed-oneplus3.zip`. See [03-rom-decision-and-flash.md](03-rom-decision-and-flash.md).
- **ROM download link** — confirmed by direct fetch of the SourceForge download page (Last Updated 2026-06-06). No longer snippet-only.
- **TWRP version** — confirmed by direct fetch of `https://dl.twrp.me/oneplus3/`: latest is `twrp-3.7.0_9-0-oneplus3.img` (2022-10-06).
- **XDA thread install instructions** — fully captured 2026-07-03 (spoiler sections pasted by user). Install steps in [03-rom-decision-and-flash.md](03-rom-decision-and-flash.md) are now verified against the OP.
- **Firmware prerequisite** — resolved: **OxygenOS 9.0.6 firmware** required; on the stock path this just means fully updating OxygenOS before unlocking. Verify with `adb shell cat /system/vendor/firmware_mnt/verinfo | grep Time_Stamp` → want `2019-11-04 21:25:29`.

## Unverified items

- **Termux:Boot reliability on the MSe1969 build** — not checked; includes whether it survives this ROM's first-boot/update cycle reliably. ([06-discord-bot-service.md](06-discord-bot-service.md))
- **FUSE support for `rclone mount` on that ROM/kernel** — not checked; expected to work since FUSE is standard on modern Android, but this is an expectation, not a confirmation. ([05-tailscale-smb.md](05-tailscale-smb.md))
- **Doze/battery-optimization exemptions on a custom ROM** — likely still needed, just less aggressive than stock; not checked. ([06-discord-bot-service.md](06-discord-bot-service.md))
- **Tailscale client's own CVE/security history** — not checked; flagged as a gap given Tailscale is now the primary trust boundary for the whole setup. ([05-tailscale-smb.md](05-tailscale-smb.md))
- **`VpnService`-based Tailscale on this specific kernel/ROM combo** — no direct reports found either way; absence of complaints isn't confirmation. ([05-tailscale-smb.md](05-tailscale-smb.md))
- **zram/swap preservation on the MSe1969 build** — architectural inference only; confirm on-device post-flash via `cat /proc/swaps` or `free -h`, as the first thing checked after flashing. ([03-rom-decision-and-flash.md](03-rom-decision-and-flash.md))
- **Firmware-update thread URL** — the OP's install instructions link a dedicated firmware thread ("THIS THREAD") and a `migration.sh` archive; **those two hyperlink URLs didn't survive the manual paste capture.** Only needed on failure paths: the firmware thread if the `verinfo` timestamp check shows pre-2019-11-04 firmware after fully updating stock OxygenOS (unlikely), and `migration.sh` only for dirty-flashing from a differently-signed build (not our route). Grab from the thread in a browser if either situation arises. ([03-rom-decision-and-flash.md](03-rom-decision-and-flash.md))
- **Battery-check sysfs paths (`charge_full` / `charge_full_design`) readable without root** — conflicting snippet-level evidence; cheap to just attempt. ([02-battery-health-check.md](02-battery-health-check.md))
- **Battery bypass (boot without battery) PMIC/kernel support on this device** — unverified; some Qualcomm designs refuse to boot without a battery. ([02-battery-health-check.md](02-battery-health-check.md))

## Dropped from scope (deliberately, not resolved)

- **OnePlus 3 bootloader relock limitation** ("only accepts test-keys, won't lock" for at least DivestOS-signed builds — earlier search snippet, never independently verified). **Dropped because it's irrelevant to this threat model:** whether the bootloader can be re-locked has zero effect on remote/network attackers (Tailscale, sshd, the bot) — those are gated by Tailscale auth regardless of lock state. The only exposure from an unlockable bootloader is that someone **physically holding the device** can plug in a cable and flash whatever they want, unsigned, no password needed. No remote exploit path opens up from this — it only matters if a person can touch the phone.
