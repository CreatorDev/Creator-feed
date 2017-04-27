include $(TOPDIR)/rules.mk


PKG_NAME:=provisioning-daemon
PKG_VERSION:=1.0.1
PKG_RELEASE:=1

PKG_SOURCE_PROTO:=git
PKG_SOURCE_URL:=https://github.com/CreatorDev/ci40-provision-daemon.git

PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.gz
PKG_SOURCE_VERSION:=$(PKG_VERSION)
PKG_SOURCE_SUBDIR:=$(PKG_NAME)-$(PKG_VERSION)

PKG_LICENSE_FILE:=LICENSE
PKG_LICENSE:=BSD-3-Clause

CMAKE_INSTALL:=1
CMAKE_OPTIONS += -DSTAGING_DIR=$(STAGING_DIR)

include $(INCLUDE_DIR)/package.mk
include $(INCLUDE_DIR)/cmake.mk

define Package/provisioning-daemon
  SECTION:=utils
  CATEGORY:=Utilities
  TITLE:=Ci40 Provisioning Daemon
  DEPENDS:=+letmecreate +ubus +ubox +libconfig +glib2 +crypto +ssl
endef

define Package/provisioning-daemon/description
  Imagination Technologies Ci40 provisioning daemon
endef

define Package/provisioning-daemon/install
	$(INSTALL_DIR) $(1)/usr/bin
	$(INSTALL_DIR) $(1)/usr/lib/lua/creator
	$(INSTALL_DIR) $(1)/etc/config
	$(INSTALL_DIR) $(1)/usr/lib/lua/luci
	$(INSTALL_DIR) $(1)/www/luci-static/resources
	$(INSTALL_BIN) $(PKG_INSTALL_DIR)/usr/bin/* $(1)/usr/bin
	$(INSTALL_DATA) $(PKG_INSTALL_DIR)/etc/config/provisioning_daemon $(1)/etc/config/provisioning_daemon
	$(CP) $(PKG_BUILD_DIR)/luci-mod-provisioning/luasrc/* $(1)/usr/lib/lua/luci
	$(CP) $(PKG_BUILD_DIR)/luci-mod-provisioning/resources/* $(1)/www/luci-static/resources
	$(CP) $(PKG_BUILD_DIR)/usr/lib/lua/creator/* $(1)/usr/lib/lua/creator

	$(INSTALL_DATA) files/etc/config/provisioning_daemon $(1)/etc/config/provisioning_daemon
endef

$(eval $(call BuildPackage,$(PKG_NAME)))
