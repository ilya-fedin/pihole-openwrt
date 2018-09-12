include $(TOPDIR)/rules.mk

PKG_NAME:=pihole
PKG_VERSION:=4.0
PKG_RELEASE:=1

PKG_SOURCE_VERSION:=ddbdb51d209fa9eea2451a1bebfaeaef23a9b77b
PKG_SOURCE:=$(PKG_SOURCE_VERSION).tar.gz
PKG_SOURCE_URL:=https://github.com/pi-hole/pi-hole/archive
PKG_HASH:=b678ce03e31bb212efe6c018153d34f1fc33c24aaee5829d0f36130b65db3fc1

PKG_LICENSE:=EUPL
PKG_LICENSE_FILES:=LICENSE

PKG_BUILD_DIR:=$(BUILD_DIR)/pi-hole-$(PKG_SOURCE_VERSION)

include $(INCLUDE_DIR)/package.mk

define Package/pihole
  SECTION:=net
  CATEGORY:=Network
  TITLE:=A black hole for Internet advertisements
  URL:=https://pi-hole.net/
  DEPENDS:=pihole-FTL +bash +bind-dig +ca-bundle +coreutils-sort +curl +idn +logrotate
  PKGARCH:=all
endef

define Package/pihole/description
  The Pi-hole® is a DNS sinkhole that protects your devices from unwanted content, without installing any client-side software.
endef

define Package/pihole/conffiles
/etc/logrotate.d/pihole
endef

define Build/Compile
endef

define Package/pihole/install
	$(INSTALL_DIR) $(1)/usr/bin
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/pihole $(1)/usr/bin/
	$(INSTALL_DIR) $(1)/opt/pihole
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/advanced/Scripts/* $(1)/opt/pihole/
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/gravity.sh $(1)/opt/pihole/
	$(INSTALL_DIR) $(1)/etc/logrotate.d
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/advanced/Templates/logrotate $(1)/etc/logrotate.d/pihole
	$(INSTALL_DIR) $(1)/etc/pihole
	echo -n "openwrt openwrt " > $(1)/etc/pihole/localbranches
	echo -n "v$(PKG_VERSION)-openwrt v$(PKG_VERSION)-openwrt v$(PKG_VERSION)-openwrt" > $(1)/etc/pihole/localversions
	touch $(1)/etc/pihole/GitHubVersions
endef

define Package/pihole/postinst
#!/bin/sh
# check if we are on real system
if [ -z "$${IPKG_INSTROOT}" ]; then
	echo "Installing cron rules"
	grep -q "pihole updateGravity" /etc/crontabs/root 2> /dev/null ||
		echo "$$((1 + RANDOM % 58)) $$((3 + RANDOM % 2)) * * 7 pihole updateGravity" >> /etc/crontabs/root
	grep -q "/usr/sbin/logrotate /etc/logrotate.conf" /etc/crontabs/root 2> /dev/null ||
		echo "@daily /usr/sbin/logrotate /etc/logrotate.conf" >> /etc/crontabs/root
	
        echo "Restarting cron"
        /etc/init.d/cron restart

	echo "Downloading and compiling the aggregated block list"
	/opt/pihole/gravity.sh --force
fi
exit 0
endef

define Package/pihole/prerm
#!/bin/sh
# check if we are on real system
if [ -z "$${IPKG_INSTROOT}" ]; then
	echo "Removing cron rules"
	grep -q "pihole updateGravity" /etc/crontabs/root 2> /dev/null &&
		sed -i "/pihole updateGravity/d" /etc/crontabs/root
	grep -q "/usr/sbin/logrotate /etc/logrotate.conf" /etc/crontabs/root 2> /dev/null &&
		sed -i "\|/usr/sbin/logrotate /etc/logrotate.conf|d" /etc/crontabs/root

        echo "Restarting cron"
        /etc/init.d/cron restart
fi
exit 0
endef

$(eval $(call BuildPackage,pihole))
