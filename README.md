# lpac-build

OpenWrt packaging + CI for our patched [lpac](https://github.com/fildunsky/lpac)
(2.3.x + AT-driver robustness PRs + FM350 fixes), so eSIM works reliably on USB
modems like the Fibocom FM350-GL via lpac's **native AT driver**.

## What this builds

`.apk` (OpenWrt 25.12.5, `apk`-based) and `.ipk` (OpenWrt 24.10.7, `opkg`-based)
for the main architectures where people run USB LTE/5G modems:

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
wrapper exports `LPAC_DRIVER_HOME=/usr/lib/lpac`.

## APDU backends

- **25.12.5** (`.apk`): **AT + QMI + UQMI + MBIM + stdio**
- **24.10.7** (`.ipk`): **AT + UQMI + MBIM + stdio** (native QMI off — see below)

- **AT** — modems that expose `AT+CCHO/CGLA` on a serial port (Fibocom FM350-GL, etc.).
- **QMI** — native QMI eUICC access via `libqmi` (through qmi-proxy, shares the
  channel with the data session). Qualcomm SDX55 modems in a QMI composition.
- **UQMI** — QMI eUICC access via the `uqmi` CLI (no glib dep). Fallback / covers
  SDX55 (Foxconn T99W175 / Thales MV31-W / Dell DW5930e) where libqmi is unavailable.
- **MBIM** — native MBIM eUICC access via `libmbim` (through mbim-proxy). For modems
  running an MBIM composition.
- **stdio** — the app's own APDU bridge over stdio.

**glib2 / native QMI+MBIM note.** libqmi/libmbim pull in glib2, whose meson build in
the OpenWrt SDK used to leak host zlib/pcre2 (`-I/usr/include`, host `libz.a` in the
target). The fix is to install glib2's target deps as feed packages (`feeds install
libqmi libmbim` pulls them). With that, native QMI+MBIM cross-build cleanly on 25.12.
**24.10 keeps native QMI OFF**: its `libqmi` is 1.34.0 but lpac's driver needs
`qmi-glib >= 1.35.5`; UQMI (the uqmi CLI, no libqmi) still covers SDX55 there. MBIM
has no such version floor and builds on both.

## Build

Push a `v*` tag (or run the workflow manually) — GitHub Actions builds every target
for both releases and, on a tag, publishes a Release with all `.apk` / `.ipk` files.

Install on the router:
- OpenWrt 25.12.x: `apk add --allow-untrusted lpac-*.apk`
- OpenWrt 24.10.x: `opkg install lpac-*.ipk`
