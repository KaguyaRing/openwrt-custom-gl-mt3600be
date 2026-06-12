#
# Pre-bundle the mihomo (Clash.Meta) core binary into the firmware,
# so OpenClash finds it on first boot without downloading it at runtime.
#
# NOTE: the mihomo binary is architecture specific. The asset below is for
# linux-arm64 (aarch64), which matches the mediatek/filogic GL-MT3600BE target.
# If you ever build for another arch, change MIHOMO_ARCH and refresh the hash.
#

include $(TOPDIR)/rules.mk

PKG_NAME:=openclash-core

MIHOMO_VERSION:=v1.19.27
MIHOMO_ARCH:=arm64

PKG_VERSION:=$(patsubst v%,%,$(MIHOMO_VERSION))
PKG_RELEASE:=1
PKG_MAINTAINER:=ChuranNeko
PKG_LICENSE:=GPL-3.0-only

# arch-specific binary inside, depends on target config
PKG_FLAGS:=nonshared

include $(INCLUDE_DIR)/package.mk

define Download/mihomo
  URL:=https://github.com/MetaCubeX/mihomo/releases/download/$(MIHOMO_VERSION)
  URL_FILE:=mihomo-linux-$(MIHOMO_ARCH)-$(MIHOMO_VERSION).gz
  FILE:=mihomo-linux-$(MIHOMO_ARCH)-$(MIHOMO_VERSION).gz
  HASH:=87db0c6660a9557a901b5750f997967e71d8c0af07ea1d1dd4d04c28da7f7e6f
endef
$(eval $(call Download,mihomo))

define Package/openclash-core
  CATEGORY:=LuCI
  SUBMENU:=3. Applications
  TITLE:=Pre-bundled mihomo (Clash.Meta) core for OpenClash
  DEPENDS:=+luci-app-openclash
  MAINTAINER:=ChuranNeko
endef

define Package/openclash-core/description
  Bundles the mihomo (Clash.Meta) core binary into the firmware so OpenClash
  works on first boot without downloading the core at runtime. Geo databases
  (GeoIP/GeoSite/Country) are updated by OpenClash itself via jsDelivr CDN.
endef

define Build/Prepare
	mkdir -p $(PKG_BUILD_DIR)
	gzip -dc $(DL_DIR)/mihomo-linux-$(MIHOMO_ARCH)-$(MIHOMO_VERSION).gz > $(PKG_BUILD_DIR)/clash_meta
endef

define Build/Compile
endef

define Package/openclash-core/install
	$(INSTALL_DIR) $(1)/etc/openclash/core
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/clash_meta $(1)/etc/openclash/core/clash_meta
endef

$(eval $(call BuildPackage,openclash-core))
