# 02 — Step 1: Battery Health Check (Optional, Zero-Risk, Run Before Flashing)

The OnePlus 3 battery is ~10 years old (2016 device). General lithium-ion calendar-aging knowledge (**not sourced this session**) suggests substantial capacity loss is likely from age alone, independent of charging habits — worth weighing battery replacement against optimizing charge behavior for an already-aged cell.

**Sequencing:** run both tests now, on current stock OxygenOS, before flashing — gives a pre-flash baseline to compare against post-flash. Zero risk, no extra tooling beyond adb (already required for the flash steps in [03-rom-decision-and-flash.md](03-rom-decision-and-flash.md)).

---

## Quick test (no root, run first)

```
adb shell dumpsys battery
```

Check:

- `health` — want `2` = GOOD; `3`/`4`/`5`/`6`/`7` are red flags worth investigating before proceeding with the rest of this plan.
- `temperature` — tenths of °C, e.g. `250` = 25.0°C.

## More precise test (root status unconfirmed — just try it)

```
adb shell cat /sys/class/power_supply/battery/charge_full
adb shell cat /sys/class/power_supply/battery/charge_full_design
```

Ratio of the two = actual capacity-fade %. **Conflicting evidence on whether this needs root on this device:**

- Search query used: `qpnp-fg oneplus3 charge_full charge_full_design sysfs battery health adb` — results are **snippets, not fetched pages**.
- One XDA guide for this exact technique is titled "[GUIDE] How To Check Battery Cycle Count and Capacity (**ROOT**)" and states root is required.
- A separate XDA-developers article on OnePlus battery health similarly says reading the equivalent path "requires special privileges" / a rooted shell.
- Against that: a UBports forum snippet shows a user reading a battery `uevent` sysfs path via plain `adb shell` with no root mentioned — permissions may be device/kernel/Android-version dependent rather than uniformly locked down.
- **Net:** cheap to just attempt — "Permission denied" means it needs root, which isn't yet a decision made in this plan (tradeoffs against the hardened ROM's security posture, see [03-rom-decision-and-flash.md](03-rom-decision-and-flash.md)).

---

## Long-term battery mitigation options

(Swelling considerations explicitly excluded per user direction.)

**Option A — Battery bypass (run off USB power only, no battery in circuit).** Strongest mitigation in principle (no battery = no degradation), but PMIC/kernel support for booting without a battery present is **unverified for this device** — some Qualcomm designs refuse to boot without one. Downsides even if it works: no power-blip buffer (hard crash risk, consider a UPS-backed outlet), and requires physically opening the adhesive-sealed glass back (real crack risk).

**Option B — Software charge-limiting via root.** Kernel-level support for charge control (`charging_enabled`/`battery_charging_enabled` sysfs attributes) is confirmed present in the msm8996 kernel family this ROM's kernel derives from (via a fetched-in-snippet comma.ai kernel fork listing) — but **known-buggy specifically on OnePlus hardware** per an XDA app-support thread, and the exact sysfs path is **not stable across ROM/kernel versions** per a separate XDA report. Requires root (Magisk), not currently part of this plan, with tradeoffs against the hardened build's security posture.

**Option C — Environmental/behavioral mitigations (apply regardless of A/B).** General li-ion knowledge, not sourced this session: heat is the dominant degradation accelerant, more so than high state-of-charge alone. Avoid enclosed/insulated mounting locations; ensure airflow.

**Net recommendation:** run the health check above first — if capacity fade is already substantial, replacing the cell outright (cheap aftermarket OP3 battery) combined with Option C is likely more effective per unit of effort than rooting the device to fight the physics of an already-decade-old cell.
