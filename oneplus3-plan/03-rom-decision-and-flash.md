# 03 — Step 2: ROM Decision (DECIDED) and Flashing Procedure

Status: **ROM and build decided; flashing not yet executed.**

---

## Chosen build

Actively-maintained unofficial LineageOS 18.1 (microG) build for oneplus3/3T by developer MSe1969 on XDA.

- Ships microG instead of full GMS, no OnePlus/OxygenOS bloat layer, minimal preinstalled apps (Aurora Store, F-Droid-style store, microG companion).
- Thread (still snippet-only — **XDA blocks automated fetching**, confirmed via a direct fetch attempt that returned a bot-detection error; opens fine in a normal browser): [xdaforums.com — [ROM][UNOFFICIAL][11.0][microG][signed] hardened LineageOS 18.1 — OnePlus 3/3T](https://xdaforums.com/t/rom-unofficial-11-0-microg-signed-hardened-lineageos-18-1-oneplus-3-3t.4347693/)
- ~~**Chosen file: `lineage-18.1-20260414-UNOFFICIAL-microG-signed-oneplus3.zip`** (April 2026 ASB patches)~~ **Superseded — thread changelog (manually captured 2026-07-03, `xda thread.txt`) lists a newer "Custom build release 2026-06-01"** (AOSmium WebView 148, F-Droid 1.23.2, AuroraStore 4.8.3). The SourceForge "Last Updated 2026-06-06" date matches the June build's upload. **Browse [SourceForge /oneplus3](https://sourceforge.net/projects/lin18-microg/files/oneplus3/) and take the 2026-06 file.**

## Why this ROM (final rationale)

**Primary reason — reliability:** stock OxygenOS on this device era has historically been aggressive about killing backgrounded processes to save battery (**general knowledge, not source-checked this session**). A custom AOSP-based ROM typically lacks that OEM-layer killer, which matters directly for the "bot must run continuously on wall power" requirement — this addresses a hard requirement, not just a soft optimization.

**Secondary reason — patch cadence:** this build still receives backported CVE fixes on a roughly monthly cadence, unlike 5-year-stale alternatives (e.g., unofficial LineageOS 17.1, EOL since ~2021). This mattered more when an HA dashboard/on-device browsing was in scope; with that scope now excluded, this is a secondary factor, not the deciding one.

**Not the deciding factor — RAM savings:** the original motivation (less bloat = more headroom) turned out to be a weak justification on its own — estimated at only a few hundred MB against a 6GB total pool, likely noise for this workload's actual footprint.

**Caveat worth flagging (critical evaluation, not just accepting the marketing):** build metadata lists `Security string: 2024-02-05` even in the April 2026 build. This means "current security patches" here means "current relative to what upstream still publishes for this legacy branch," not "2026 Android security standard." Likely explained by Google no longer issuing official ASB bulletins for this branch — individual CVE fixes are still being cherry-picked and backported monthly, just without a fresh official security-string bump to point to. Irrelevant to the bot/Tailscale/Pi-hole workload; could matter only if an installed app checks patch level.

---

## Confirmed facts: download link, TWRP version, zram

**ROM download link and currency — confirmed by direct fetch (not a snippet).**
Fetched `https://sourceforge.net/projects/lin18-microg/files/oneplus3/lineage-18.1-20260414-UNOFFICIAL-microG-signed-oneplus3.zip/download` directly. The page loaded (SourceForge, not blocked), confirming:

- Project: `lin18-microG`, maintainer `mse1969` (matches XDA username MSe1969).
- **Last Updated: 2026-06-06** — matches the XDA forum-index date from earlier snippets, now independently corroborated by a fetched primary source.
- Registered 2021-08-17 (project has been running ~5 years).
- The file is a real, live download — page returned "Your download will start shortly," not a 404/removed notice.
- Could not fetch the XDA thread itself — `xdaforums.com` returned a bot-detection error on direct fetch (`CLIENT_ERROR: Site blocked the request`). All XDA-sourced details (install steps, "don't flash GApps," feature list) remain **snippet-only**, not independently fetched/verified.

**TWRP version — confirmed by direct fetch of the official TWRP page.**
Fetched `https://dl.twrp.me/oneplus3/` directly. Full version list retrieved; the current latest is:

- **`twrp-3.7.0_9-0-oneplus3.img`**, 29.1MB, dated **2022-10-06 08:00:08 UTC**.
- This is the newest of 23 listed builds going back to 2016; no newer official TWRP has been released for this device since October 2022 (consistent with the device's age and shrinking maintainer interest — general inference, not stated on the page itself).
- The `_9-0` in the filename denotes the Android-Pie-era tree TWRP is built against; per the general (non-device-specific) TWRP installation page fetched earlier, this is standard and expected to still work across the Oreo/Pie/Q+ custom ROMs commonly flashed on this device — TWRP itself doesn't need to match your target ROM's Android version.
- **PGP key available for verification**: `https://dl.twrp.me/public.asc` — recommended given you're downloading a nearly-4-year-old binary from a device the official maintainer has likely moved on from.

**zram/swap preservation — NOT independently confirmed. Architectural inference only.**
Search query used: `oneplus3 lineageos zram swap msm8996 kernel default` — result is a **search snippet, not a fetched page**, dated 2017 (an XDA thread titled "Disabling ZRAM for msm8996 targets"), not from the current MSe1969 build specifically.

- That thread describes LineageOS enabling zram unconditionally for msm8996-family devices at the kernel/fstab level: `rootdir/root/fstab.qcom: +/dev/block/zram0 none swap defaults` and `init.qcom.rc` setting `zram0/comp_algorithm` to `lz4`.
- **Reasoning, not verification:** the MSe1969 18.1 build is described (per earlier snippets) as based on an "upstreamed" LineageOS kernel with patches layered on top, not a from-scratch kernel rewrite. It's therefore plausible this fstab/init-level zram config is inherited unchanged, since it's a platform-level default rather than a device-specific feature someone would likely strip out. **This is an estimate based on how LineageOS-family kernels are typically structured, not a confirmed fact about this specific build.**
- To actually confirm: after flashing, run `cat /proc/swaps` or `free -h` on-device — this settles it definitively in seconds and should be the first thing checked post-flash.

---

## Before flashing — thread review status (updated 2026-07-03 from manual capture, `xda thread.txt`)

The thread's first page was manually captured. Confirmed from it:

- **Clean flash required** when coming from a differently-signed build (i.e., from OxygenOS): the OP says you cannot dirty-flash across signing keys. The wipe steps below already comply.
- **Firmware prerequisite is real:** the maintainer states the latest OnePlus firmware for this device is Android-9-era, a user references "the debloated firmware linked in the OP," and the credits thank nvertigo67 "for the modded 9.x firmware." A specific firmware package is linked in the OP.
- **Security string 2024-02-05** confirmed verbatim; kernel patched from Google's `android-4.14-q-release` branch; no Android 12+ will ever come (LineageOS legacy-device policy, deliberate).

**Spoiler sections now captured (user pasted them 2026-07-03) — install steps below are verified against the OP.** New facts from the spoilers:

- **Firmware prerequisite pinned down: OxygenOS 9.0.6 firmware is required for LineageOS 18.1** (latest firmware OnePlus ever published for this device is Android-9-era). On our path — coming from stock OxygenOS — the OP says simply: **update to the latest offered OxygenOS version before starting** ("if not [offered], no issue"), which yields the correct firmware. No separate firmware flash needed from stock.
- **Firmware verification command** (for custom-ROM situations, or to double-check): `adb shell cat /system/vendor/firmware_mnt/verinfo | grep Time_Stamp` as root — latest is `"Time_Stamp": "2019-11-04 21:25:29"`; anything earlier means the firmware needs updating first via a separate firmware thread. The OP links that thread as "THIS THREAD" — **the URL didn't survive the paste capture**; only needed if the timestamp check fails, and it shouldn't on fully-updated stock.
- The OP "strongly recommends" the **debloated firmware** (re: the Alipay/WeChatpay/Soter/IFAA section of the firmware thread) — optional consideration; the ROM itself is already "debloated from Oneplus blobs for Soter and IFAA."
- **"Dealing with signed builds" / `migration.sh` is irrelevant to us** — it's only for dirty-flashing from a differently-signed build. We clean-flash from stock, which is the OP-recommended route anyway. (The `migration.sh` archive link also didn't survive the paste; not needed.)
- Source links from the OP: [kernel](https://github.com/lin18-microg/android_kernel_oneplus_msm8996/tree/lin-18.1-mse3), [build manifest](https://github.com/lin18-microg/local_manifests/tree/lin-18.1-hmalloc), [why no Android 12+](https://lineageos.org/Changelog-26/) (section "Let's talk about legacy devices…").

**Post-install gotchas (from thread posts #10–11):**

- The ROM's Updater app **replaces TWRP with LineageOS recovery on OTA updates** unless a setting in the Updater app is switched off — toggle it after first boot if you want to keep TWRP.
- OTA support means monthly unattended reboots — each one re-tests Termux:Boot (see [06-discord-bot-service.md](06-discord-bot-service.md)).

## Install steps (Windows 11 PC side) — verified against the OP's installation instructions

1. **On stock OxygenOS, before anything else: update to the latest offered OxygenOS version** (Settings → System Update, repeat until nothing more is offered). This is the OP's firmware prerequisite for stock users — it brings the device to the required **OxygenOS 9.0.6 firmware**. Do this *before* unlocking, since unlocking wipes the device.
2. Install Android SDK platform-tools (adb/fastboot) on Windows 11. Built-in WinUSB support generally works for fastboot; if `fastboot devices` shows nothing, install the OnePlus USB driver or bind WinUSB via Zadig. (The OP explicitly doesn't support PC-side USB troubleshooting — search XDA if stuck.)
3. Download the most current ROM `.zip` (2026-06 build, see above) — the OP says to place it on the phone's internal memory; doing it now (while Android is booted, plain MTP) is easiest, **but note step 4 wipes data — internal storage survives the unlock wipe on some devices but not others, so be prepared to re-copy it at step 7.**
4. On phone: enable Developer Options (tap Build Number 7x) → enable USB Debugging + OEM Unlocking. Then `adb reboot bootloader` → `fastboot oem unlock` — **wipes all data. Back up first.**
5. `fastboot flash recovery twrp-3.7.0_9-0-oneplus3.img`, then boot **directly into recovery**: `fastboot reboot` while holding **Power + Vol-down** (or `fastboot reboot recovery`). **The first boot after flashing TWRP must be TWRP** — do not let it boot into Android, or stock firmware may overwrite TWRP.
6. In TWRP: Wipe → Advanced → check **Dalvik, System, Cache, Data**. **Do NOT wipe Internal Memory.** Swipe to confirm. (OP: this wipe is only for people coming from stock or a different ROM — that's us.)
7. If the ROM zip didn't survive: copy it to internal storage now (MTP works inside TWRP, or `adb push <rom>.zip /sdcard/`).
8. TWRP main menu → Install → navigate to `/sdcard` → select the ROM zip → swipe to flash. **Do NOT flash GApps** — microG is pre-installed and conflicts.
9. Reboot → System. **First boot after flashing takes quite long** (OP's words) — several minutes is expected, not a hang.
10. Immediately check `cat /proc/swaps` or `free -h` to resolve the open zram question above.
11. In the ROM's Updater app: disable the setting that replaces TWRP with LineageOS recovery on OTA updates (if you want to keep TWRP — see post-install gotchas above).

> OP disclaimer, worth keeping verbatim in spirit: "YOU ARE RESPONSIBLE SOLELY YOURSELF FOR ANY ACTIONS YOU DO WITH YOUR DEVICE."

> ROM note from repo README: the setup script works on stock Android too — the video demonstrates it running on **LineageOS** on a OnePlus 5T. Rooting is not required by the script itself.

---

## Reasoning history (how the decision evolved — kept for reference)

1. **Original driver:** wanted "less bloat = more RAM for a Linux desktop in Termux." Assessed as a real but likely modest effect — Android 11 (this ROM's base) has a heavier baseline framework footprint than Android 8/9 stock, partially offsetting the bloat savings. **Estimate, not measured.**
2. **Security check (WebView CVE research):** confirmed via 2026 search results (multiple CVEs: 2026-11007, 2026-10967, 2026-11097, 2026-0628, 2026-3909 — high/critical severity, some requiring only a page visit, one confirmed actively exploited in the wild) that Chrome/WebView patches land roughly monthly in 2026. This argued _against_ a 5-year-stale ROM **if** the device would render untrusted web content.
3. **Scope correction:** user confirmed no HA dashboard, no stated on-device browsing. **This substantially weakened the WebView-CVE argument** — none of Tailscale, the Discord bot, or Pi-hole render untrusted web content.
4. **Revised RAM assessment:** the actual planned workload (tailscaled, sshd, a Discord bot process, optionally Pi-hole's pihole-FTL + dnsmasq/unbound) is architecturally lightweight — **estimated 200–400MB combined, not measured** — against a 6GB total pool.
5. **Reliability argument** (became the primary reason — see final rationale above).
