# SPDX-License-Identifier: AGPL-3.0-only
#
# lpac 2.3.x + FM350 patches, packaged for OpenWrt.
# Source is our patched fork (estkme-group/lpac main + curated PRs + the
# LPAC_DRIVER_HOME loader patch), pulled by git at the v2.3.0-fm350 tag.
#
# 2.3.x split lpac into shared libs + dlopen'd driver plugins found via ELF
# RUNPATH; OpenWrt strips RUNPATH (CMAKE_SKIP_RPATH), so our loader patch adds a
# getenv("LPAC_DRIVER_HOME") fallback and we ship the plugins under
# /usr/lib/lpac/driver, with the wrapper exporting LPAC_DRIVER_HOME=/usr/lib/lpac.

include $(TOPDIR)/rules.mk

PKG_NAME:=lpac
PKG_VERSION:=2.3.0_p4
PKG_RELEASE:=1

PKG_SOURCE_PROTO:=git
PKG_SOURCE_URL:=https://github.com/fildunsky/lpac.git
PKG_SOURCE_VERSION:=v2.3.0-fm350
PKG_MIRROR_HASH:=skip

PKG_MAINTAINER:=fildunsky
PKG_LICENSE:=AGPL-3.0-only LGPL-2.0-only

# libqmi/libmbim нужны ТОЛЬКО нативным QMI/MBIM-бэкендам (qmi.c/mbim.c линкуют
# qmi-glib/mbim-glib). Форсим их стейджинг ЛИШЬ когда соответствующий бэкенд
# включён - иначе SDK впустую собирал бы glib2 (в CI он ловит host-header leak
# "-I/usr/include" -> bits/libc-header-start.h) для сборки, где нативный QMI/MBIM
# не нужен. UQMI-бэкенд вызывает uqmi CLI в рантайме, glib и стейджинга не требует,
# поэтому AT+UQMI собирается без glib2 вовсе.
PKG_BUILD_DEPENDS:=$(if $(CONFIG_LPAC_WITH_QMI),libqmi) $(if $(CONFIG_LPAC_WITH_MBIM),libmbim)

CMAKE_INSTALL:=1

include $(INCLUDE_DIR)/package.mk
include $(INCLUDE_DIR)/cmake.mk

define Package/lpac
  SECTION:=utils
  CATEGORY:=Utilities
  TITLE:=eUICC eSIM LPA (2.3.x + FM350 patches)
  DEPENDS:= \
    +LPAC_WITH_PCSC:libpcsclite \
    +LPAC_WITH_PCSC:pcscd \
    +LPAC_WITH_QMI:libqmi \
    +LPAC_WITH_MBIM:libmbim \
    +LPAC_WITH_UQMI:uqmi
  URL:=https://github.com/fildunsky/lpac
endef

define Package/lpac/description
  lpac is a eUICC eSIM LPA manager written in C. This build tracks our patched
  fork (2.3.x + AT-driver robustness PRs #446/#447/#448, FM350 bare-CCHO #449,
  and the LPAC_DRIVER_HOME loader fix). It ships every modem APDU backend - AT
  (Fibocom FM350-GL), QMI (libqmi) and uqmi plus MBIM (Qualcomm SDX55 modems
  such as Foxconn T99W175 / Thales MV31-W / Dell DW5930e) - and the stdio bridge
  for HTTP, so the LuCI app can pick the right transport per modem.
endef

define Package/lpac/config
  source "$(SOURCE)/Config.in"
endef

define Package/lpac/conffiles
/etc/config/lpac
endef

TARGET_CFLAGS += $(FPIC)

CMAKE_OPTIONS += \
  -DLPAC_WITH_APDU_PCSC=$(if $(CONFIG_LPAC_WITH_PCSC),ON,OFF) \
  -DLPAC_WITH_APDU_AT=$(if $(CONFIG_LPAC_WITH_AT),ON,OFF) \
  -DLPAC_WITH_APDU_QMI=$(if $(CONFIG_LPAC_WITH_QMI),ON,OFF) \
  -DLPAC_WITH_APDU_UQMI=$(if $(CONFIG_LPAC_WITH_UQMI),ON,OFF) \
  -DLPAC_WITH_APDU_MBIM=$(if $(CONFIG_LPAC_WITH_MBIM),ON,OFF) \
  -DLPAC_WITH_APDU_QMI_QRTR=OFF \
  -DLPAC_WITH_APDU_GBINDER=OFF \
  -DLPAC_WITH_HTTP_CURL=OFF

define Package/lpac/install
	$(INSTALL_DIR) $(1)/usr/bin
	$(INSTALL_BIN) ./files/lpac.sh $(1)/usr/bin/lpac

	$(INSTALL_DIR) $(1)/etc/config
	$(CP) ./files/lpac.uci $(1)/etc/config/lpac

	# Real binary + dlopen'd driver plugins under /usr/lib/lpac; the loader finds
	# the plugins via LPAC_DRIVER_HOME (our patch), since OpenWrt strips RUNPATH.
	$(INSTALL_DIR) $(1)/usr/lib/lpac
	$(INSTALL_BIN) $(PKG_INSTALL_DIR)/usr/bin/lpac $(1)/usr/lib/lpac/lpac
	$(INSTALL_DIR) $(1)/usr/lib/lpac/driver
	$(CP) $(PKG_INSTALL_DIR)/usr/lib/lpac/driver/*.so $(1)/usr/lib/lpac/driver/
	$(INSTALL_DIR) $(1)/usr/lib
	$(CP) $(PKG_INSTALL_DIR)/usr/lib/lib*.so* $(1)/usr/lib/
endef

$(eval $(call BuildPackage,lpac))
