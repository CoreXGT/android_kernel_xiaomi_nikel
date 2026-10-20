# Provenance of this kernel tree

This document records where the code in this tree came from, so that anyone
picking it up does not have to repeat the investigation.

## The short version

The tree is **not** a GitHub fork of another repository. It is an independent
upload, seeded from a **MediaTek MT6797 platform kernel** that had already
been partially adapted to `nikel` by a third party, and then developed on top
of that by the original author.

## Evidence

### 1. Not a GitHub fork

The upstream repository reports `fork: false` with no parent repository, and
its first commit (`ebd1d87`, 19 Oct 2021, "Nikel: init ported source") is a
single wholesale import of the whole tree by `ruben1863`. The word "ported" in
that commit message is the tell: the code was brought in from outside.

All 14 commits on `master` are the original author's own work. The only
outside contributions live on the `develop` branch (3 commits by `jmpfbmx`,
Aug 2023).

### 2. Not from Xiaomi's GPL release

Xiaomi publishes GPL kernel sources through
[MiCode/Xiaomi_Kernel_OpenSource](https://github.com/MiCode/Xiaomi_Kernel_OpenSource).
That repository has **no `nikel` branch** — the branch list goes from `nabu`
straight to `nitrogen`. Xiaomi never released a GPL kernel for the Note 4 MTK.

### 3. Closely related to the Meizu MX6 (MT6797) platform release

The Meizu MX6 GPL release (mirrored at
[8osm/android_kernel_xiaomi_nikel](https://github.com/8osm/android_kernel_xiaomi_nikel),
first commit "Meizu OpenSource", 2016) is the closest publicly available
relative of this tree. Comparing the two by git blob hash:

| Measurement | Result |
| --- | --- |
| Files in Ruben's initial import | 53,682 |
| Files in the Meizu release | 54,034 |
| Common paths | 53,422 |
| **Byte-identical content** | **53,052 (99.3%)** |
| Differing content | 370 |
| Present only in Ruben's import | 260 |
| Present only in the Meizu release | 612 |

The 370 differing files are precisely the adapter layer: the device tree,
custom MTK `cust_*.dtsi` headers, defconfigs and board files. The 260 files
unique to Ruben's tree are the `cust_*.dtsi` headers, `k15ta_a` / `k15tb_a` /
`nb6797_6c_m` board files and the `nikel_defconfig`. The 612 files unique to
the Meizu release are the Meizu boards themselves (`meizu6797_5t.dts`,
`meizu6797_5u.dts`, `meizu6797_6c_m.dts`, `mx6_defconfig`, Meizu sensor
drivers).

Both trees are kernel **3.18.22** with the full MediaTek vendor platform
(`Android.mk`, `drivers/misc/mediatek/mach/mt6797/`, `drivers/misc/mediatek/lcm/`).

### 4. The nikel-specific work is the original author's

At the initial import there was no `nikel.dts` — only the other ODM project
device trees (`k15ta_a`, `k15tb_a`, `nb6797_6c_m`). A `nikel_defconfig`
existed, but the device tree did not. The author created `arch/arm64/boot/dts/nikel.dts`
himself in commit `8369a75` ("Nikel: clean tree and make minor changes",
19 Jan 2022) and then worked on the LCM, touch and boot support from there.

The `nikel.dts` in this tree is a descendant of the `nb6797_6c_m.dts` adapter
(the two share the same memory map and `i2c0` layout), with the panel and
charger nodes adjusted for nikel.

## What this fork changed

The `dev` branch is based on Ruben's `backup-omen` branch, which was the most
complete state upstream had (Nov 2023). That branch, however, **could not
compile**: an interrupted `git stash` had left unresolved conflict markers
committed in five files, and several defconfig entries were malformed.

| Change | Reason |
| --- | --- |
| Resolved conflict markers in `arch/arm64/boot/dts/mt6797.dtsi`, `arch/arm64/boot/dts/nikel.dts`, `drivers/misc/mediatek/power/mt6797/lct_bq24296.c`, `drivers/power/mediatek/battery_common.c`, `drivers/power/mediatek/battery_meter.c` | The upstream branch shipped them unresolved; the tree did not build at all. |
| Kept the `cw2015@0` and `sw_charger@0` device tree nodes | The `cw2015` battery driver and the `BQ24296` charger driver probe for them; dropping them silently disables both. |
| Removed the duplicate definition of `FG_charging_status` from `battery_common.c` | It is already defined in `focaltech_core.c`; keeping both is a duplicate-symbol link error. |
| Removed `*.bak` files and the `k15ta_a` / `k15tb_a` / `nb6797_6c_m-old` board directories | Dead weight from the source tree; not referenced by any Makefile or Kconfig. |
| Fixed `#CONFIG_MTK_BQ25896_SUPPORT=y` → `CONFIG_MTK_BQ25896_SUPPORT=n` and `CONFIG_MTK_BQ24296_SUPPORT=n` → `=y` in `nikel_defconfig` | Nikel uses the BQ24296 (I2C address `0x6b`, see `nikel.dts`). The commented-out form was a no-op, so the tree was building the wrong charger driver. `nikel_defconfig` now matches `nikel_debug_defconfig`. |
| Fixed `#CONFIG_MTK_ICM20645G=y` and the other five sensor entries to real `CONFIG_...=y` lines | A line starting with `#` is a comment, not a disabled option, so all six sensors were silently off even though `nikel.dts` declares exactly those six sensor nodes. |
| Fixed `#CONFIG_SND_SOC_MAX98926=y` → `CONFIG_SND_SOC_MAX98926=y` | The speaker amplifier codec, whose `max98926L@31` node is declared in `nikel.dts`. |
| Added `.gitignore` for `/out/`, `/gcc/` and build logs | The toolchain and build output were previously untracked clutter. |

## Practical consequence

Because the tree is 99.3% identical to the Meizu MX6 GPL release, that
release is a convenient reference when a driver needs to be compared against
its pre-port state. It is **not** a better development base: it contains no
`nikel` device tree, no `nikel` defconfig and none of the LCM, touch or
charger work that makes the device boot.
