# Bugfixes

A running list of the fixes applied on top of Ruben's `backup-omen` branch,
which was the upstream state this fork started from.

The upstream `backup-omen` branch **did not compile at all**. Every fix below
was required to get a working `Image.gz-dtb` out of the tree.

## Build blockers (the tree did not build)

### 1. Unresolved conflict markers in five files

An interrupted `git stash` had left conflict markers committed into the
branch. The tree could not be compiled, let alone booted.

* `arch/arm64/boot/dts/mt6797.dtsi`
* `arch/arm64/boot/dts/nikel.dts`
* `drivers/misc/mediatek/power/mt6797/lct_bq24296.c`
* `drivers/power/mediatek/battery_common.c`
* `drivers/power/mediatek/battery_meter.c`

Resolved by hand. Where both sides were equivalent, the upstream side was
kept. The `cw2015@0` and `sw_charger@0` device tree nodes were kept because
the `cw2015` fuel gauge driver and the `BQ24296` charger driver probe for
them — dropping them silently disables both drivers.

### 2. Duplicate symbol `FG_charging_status`

`FG_charging_status` was defined in both
`drivers/input/touchscreen/mediatek/ft5346/focaltech_core.c` and
`drivers/power/mediatek/battery_common.c`, which is a link error.

The definition in `focaltech_core.c` was kept and the one in
`battery_common.c` was removed, with a comment pointing at the real owner.

### 3. Missing definition of `FG_charging_type`

`drivers/misc/mediatek/battery_fual/cw2015_battery.c` declares
`extern int FG_charging_type` and uses it, but nothing defined it anywhere in
the tree — an undefined reference at link time.

The definition was restored in
`drivers/misc/mediatek/power/mt6797/pmic_chr_type_det.c` (which is always
built, `CONFIG_MTK_PMIC` defaults to `y`), together with the assignment in
`hw_charging_get_charger_type()` so the value actually tracks the detected
charger type. This matches what the `develop` branch had before the driver
was moved.

### 4. `charge_suspend` defined outside its scope

`bool charge_suspend` was defined inside the
`#if defined(CONFIG_MTK_PUMP_EXPRESS_PLUS_SUPPORT)` block in
`drivers/power/mediatek/switch_charging.c`, but it is also used in
`hw_charging_get_current()`, which sits outside that block. With Pump
Express disabled the symbol was missing and the file failed to compile.

The definition was moved out of the conditional block, next to the other
charger globals.

### 5. Stale include of the renamed charger header

`drivers/misc/mediatek/xhci/xhci-mtk-driver.c` still included
`<rn4x_bq24296.h>`, but the file had been renamed to `lct_bq24296.h`. Every
symbol it needs is declared in the new header, so only the include was
updated.

## Defconfig corrections

### 6. Wrong charger driver selected

`nikel_defconfig` had

```
#CONFIG_MTK_BQ25896_SUPPORT=y
CONFIG_MTK_BQ24296_SUPPORT=n
```

A line starting with `#` is a comment, not a disabled option, so this
actually enabled the **BQ25896** driver and disabled the **BQ24296** one.

Nikel uses the BQ24296, wired at I2C address `0x6b` (see
`arch/arm64/boot/dts/nikel.dts`). Corrected to:

```
CONFIG_MTK_BQ24296_SUPPORT=y
CONFIG_MTK_BQ25896_SUPPORT=n
```

`nikel_defconfig` now matches `nikel_debug_defconfig` on this point.

### 7. All six sensors were silently disabled

The sensor entries used the same broken format:

```
#CONFIG_MTK_ICM20645G=y
#CONFIG_MTK_CM36558=y
#CONFIG_MTK_ICM20645GY=y
#CONFIG_MTK_AKM09912=y
#CONFIG_MTK_BMP280=y
#CONFIG_MTK_HTS221=y
```

`nikel.dts` declares exactly these six sensor nodes
(`cust_accel@0`, `cust_alsps@0`, `cust_gyro@0`, `cust_mag@0`,
`cust_barometer@0`, `cust_humidity@0`), so the intent was clearly to enable
them. All six were rewritten as real `CONFIG_...=y` lines in both
defconfigs.

### 8. Speaker amplifier codec disabled

`#CONFIG_SND_SOC_MAX98926=y` had the same problem, while `nikel.dts`
declares the `max98926L@31` node. Enabled in both defconfigs.

## Housekeeping

* Removed three leftover `*.bak` files.
* Removed the `k15ta_a`, `k15tb_a` and `nb6797_6c_m-old` board directories
  left over from the source tree. None of them were referenced by any
  Makefile or Kconfig.
* Added a `.gitignore` for `/out/`, `/gcc/` and the build logs.
* Added `PROVENANCE.md` documenting where the code came from.
