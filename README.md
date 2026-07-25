# lpac-build

OpenWrt packaging + CI for our patched [lpac](https://github.com/fildunsky/lpac)
(2.3.x + AT-driver robustness PRs + FM350 fixes), so eSIM works reliably on USB
modems like the Fibocom FM350-GL via lpac's **native AT driver**.

## What this builds

`.apk` (OpenWrt 25.12.5, `apk`-based) for the main architectures where people run
USB LTE/5G modems:

| Target | Arch | Typical devices |
|---|---|---|
| mediatek/filogic | aarch64_cortex-a53 | WH3000 & new WiFi6 routers with USB |
| rockchip/armv8 | aarch64 | NanoPi R2S/R4S/R5S |
| bcm27xx/bcm2711 | aarch64_cortex-a72 | Raspberry Pi 4 |
| armsr/armv8 | aarch64_generic | VMs / containers / generic ARM64 |
| armsr/armv7 | arm | generic ARM32 |
| ramips/mt7621 | mipsel_24kc | Xiaomi / GL.iNet / Netgear |
| ath79/generic | mips_24kc | older MIPS routers with USB |
| x86/64 | x86_64 | mini-PC / VM routers |

## Source

The lpac source is pulled by git from `fildunsky/lpac` at tag `v2.3.0-fm350`
(estkme-group/lpac main + curated PRs #422/#443/#446/#447/#448/#449/#451/#452/#453
+ a `LPAC_DRIVER_HOME` loader patch). Change `PKG_SOURCE_VERSION` in
`openwrt/Makefile` to build a different tag.

## Packaging notes

2.3.x splits lpac into shared libs + dlopen'd driver plugins found via ELF
RUNPATH, which OpenWrt strips. So: the loader honors `LPAC_DRIVER_HOME`, plugins
ship under `/usr/lib/lpac/driver/`, libs under `/usr/lib/`, and the `/usr/bin/lpac`
wrapper exports `LPAC_DRIVER_HOME=/usr/lib/lpac`. Backends: AT + stdio + curl
(PCSC/UQMI/MBIM/QMI/GBinder off).

## Build

Push a `v*` tag (or run the workflow manually) — GitHub Actions builds all targets
and, on a tag, publishes a Release with the `.apk` files.

Install on the router: `apk add --allow-untrusted lpac-*.apk`.
