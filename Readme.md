# Custom kernel for Xiaomi Redmi Note 4 MTK (Nikel) by Ruben1863
# Kernel version 3.18.22

A clean, buildable development baseline for the Xiaomi Redmi Note 4 **MTK**
variant (codename `nikel`, SoC MediaTek MT6797 / Helio X20).

This tree is a fork of [@Ruben1863/android_kernel_xiaomi_nikel](https://github.com/Ruben1863/android_kernel_xiaomi_nikel).
See [PROVENANCE.md](PROVENANCE.md) for the full history of where the code
came from and what was changed in this fork.

## Branch layout

| Branch | Base | Purpose |
| --- | --- | --- |
| `master` | `origin/master` (Ruben) | Upstream mirror, unmodified. Use as the reference for the last known-good "boots with LCM + touch" state. |
| `dev` | `origin/backup-omen` (Ruben) | **Active development branch.** Contains the most complete hardware support upstream had, plus the sanitization commits described below. |
| `origin/develop` | (Ruben) | Kept for reference only. Its charging rewrite is marked "not working" by its own author and it conflicts with `backup-omen`. Not a build base. |

## Hardware status

Working:

* LCM `nt35532_fhd_boe_vdo_lcm` (BOE)
* LCM `nt35596_fhd_tianma_phantom` (Tianma)
* Touch (FT5346)
* Boot, MTP, ADB
* Battery gauge `cw2015` (`CONFIG_FG_CW2015_BATTERY_SUPPORT`)
* Charger `BQ24296` (`CONFIG_MTK_BQ24296_SUPPORT`)

Not working / unverified:

* Almost everything else, in particular: camera, WiFi/BT, GPS, audio
  playback and calls. Treat this as a bring-up tree, not a daily driver.

> **Note on panels and sensors:** the Redmi Note 4 MTK shipped with several
> panel and sensor combinations. The two LCMs above are the only ones ported.
> If your unit shows a black screen, that is almost certainly an unsupported
> panel, not a broken build. Check `/proc/lcm_name` on a running device and
> port the matching LCM driver.

## Building

The tree builds with the AOSP GCC 4.9 toolchain (MT6797 vendor kernels are
not clang-clean). The `Build` script fetches the toolchain for you:

```bash
chmod a+x Build && bash Build
```

Manual equivalent:

```bash
git clone --depth=1 -b marshmallow-release \
    https://android.googlesource.com/platform/prebuilts/gcc/linux-x86/aarch64/aarch64-linux-android-4.9 gcc
mkdir -p out
export ARCH=arm64
export CROSS_COMPILE=${PWD}/gcc/bin/aarch64-linux-android-
make O=out ARCH=arm64 CROSS_COMPILE=$CROSS_COMPILE nikel_defconfig
make O=out ARCH=arm64 CROSS_COMPILE=$CROSS_COMPILE -j$(nproc)
```

The resulting image is `out/arch/arm64/boot/Image.gz-dtb` (the DTB is appended
to the kernel, per `CONFIG_BUILD_ARM64_APPENDED_DTB_IMAGE`).

### Defconfigs

* `nikel_defconfig` — release configuration.
* `nikel_debug_defconfig` — adds KMEMLEAK, LOCKDEP, PROVE_LOCKING, KGDB,
  ftrace and other debug options. Use it to chase crashes, not for daily use.

## Flashing

This is a standalone kernel image. To test it without touching the ROM,
unpack a boot image (e.g. the one from your current ROM, or TWRP) with
`magiskboot`/AnyKernel3, replace `Image.gz-dtb`, repack and flash to the
`boot` partition:

```bash
fastboot flash boot boot.img
```

Keep a known-good boot image handy — the display is the first thing that
breaks when hardware support is incomplete.

## Acknowledgements

* [@Ruben1863](https://github.com/Ruben1863) — original kernel port for nikel
* [@jmpfbmx](https://github.com/jmpfbmx) — `nikel_debug_defconfig`, build script
* [@R0rt1z2](https://github.com/R0rt1z2)
* MediaTek — the MT6797 platform kernel this tree is derived from
* Meizu — the GPL release of the closely related MX6 (MT6797) platform tree,
  used as an upstream reference during this fork's cleanup
