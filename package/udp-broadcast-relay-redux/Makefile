include $(TOPDIR)/rules.mk

PKG_NAME:=udp-broadcast-relay-redux
PKG_VERSION:=1.0.0
PKG_RELEASE:=1

PKG_SOURCE_PROTO:=git
PKG_SOURCE_URL:=https://github.com/udp-redux/udp-broadcast-relay-redux
PKG_SOURCE_VERSION:=HEAD
PKG_MIRROR_HASH:=skip
PKG_LICENSE:=GPL-2.0-only
PKG_LICENSE_FILES:=LICENSE
PKG_MAINTAINER:=Paolo Antinori <paolo.antinori@gmail.com>

PKG_BUILD_DEPENDS:=

include $(INCLUDE_DIR)/package.mk

define Package/udp-broadcast-relay-redux
  SECTION:=net
  CATEGORY:=Network
  TITLE:=UDP broadcast and multicast relay across interfaces
  URL:=https://github.com/udp-redux/udp-broadcast-relay-redux
  DEPENDS:=
  PKGARCH:=all
endef

define Package/udp-broadcast-relay-redux/description
  Relays UDP broadcast and multicast packets between interfaces (e.g.
  across VLANs or subnets) preserving the original source address via
  raw sockets, so replies reach the real sender. Configured through
  UCI (/etc/config/udp-broadcast-relay-redux), one procd instance per
  relay section.
endef

define Build/Prepare
	mkdir -p $(PKG_BUILD_DIR)
	$(CP) ./src/* $(PKG_BUILD_DIR)/
endef

define Build/Compile
	$(TARGET_CC) $(TARGET_CFLAGS) -o $(PKG_BUILD_DIR)/udp-broadcast-relay-redux $(PKG_BUILD_DIR)/main.c
endef

define Package/udp-broadcast-relay-redux/conffiles
/etc/config/udp-broadcast-relay-redux
endef

define Package/udp-broadcast-relay-redux/install
	$(INSTALL_DIR) $(1)/usr/bin
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/udp-broadcast-relay-redux $(1)/usr/bin/
	$(INSTALL_DIR) $(1)/etc/config
	$(INSTALL_CONF) ./files/etc/config/udp-broadcast-relay-redux $(1)/etc/config/
	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) ./files/etc/init.d/udp-broadcast-relay-redux $(1)/etc/init.d/
endef

$(eval $(call BuildPackage,udp-broadcast-relay-redux))
