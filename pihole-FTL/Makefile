include $(TOPDIR)/rules.mk

PKG_NAME:=pihole-FTL
PKG_VERSION:=4.0
PKG_RELEASE:=1

PKG_SOURCE_VERSION:=8493df476758650e407d6be83321500fba81f7d6
PKG_SOURCE:=$(PKG_SOURCE_VERSION).tar.gz
PKG_SOURCE_URL:=https://github.com/pi-hole/FTL/archive
PKG_HASH:=8afa849f46c9cdf25ba068ebf816e7bfd4299c2a59505ecad4adb8831a362215

PKG_LICENSE:=EUPL
PKG_LICENSE_FILES:=LICENSE

PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)-$(BUILD_VARIANT)/FTL-$(PKG_SOURCE_VERSION)

PKG_INSTALL:=1
PKG_BUILD_PARALLEL:=1
PKG_CONFIG_DEPENDS:= CONFIG_PACKAGE_pihole-FTL_$(BUILD_VARIANT)_dhcp \
	CONFIG_PACKAGE_pihole-FTL_$(BUILD_VARIANT)_dhcpv6 \
	CONFIG_PACKAGE_pihole-FTL_$(BUILD_VARIANT)_dnssec \
	CONFIG_PACKAGE_pihole-FTL_$(BUILD_VARIANT)_auth \
	CONFIG_PACKAGE_pihole-FTL_$(BUILD_VARIANT)_ipset \
	CONFIG_PACKAGE_pihole-FTL_$(BUILD_VARIANT)_conntrack \
	CONFIG_PACKAGE_pihole-FTL_$(BUILD_VARIANT)_noid \
	CONFIG_PACKAGE_pihole-FTL_$(BUILD_VARIANT)_broken_rtc

include $(INCLUDE_DIR)/package.mk

define Package/pihole-FTL/Default
  SECTION:=net
  CATEGORY:=Network
  TITLE:=The Pi-hole FTL engine
  URL:=https://pi-hole.net/
  DEPENDS:=+libubus
  CONFLICTS:=dnsmasq dnsmasq-dhcpv6 dnsmasq-full
  PROVIDES:=dnsmasq
  USERID:=pihole:pihole
endef

define Package/pihole-FTL
$(call Package/pihole-FTL/Default)
  VARIANT:=nodhcpv6
  CONFLICTS+=pihole-FTL-dhcpv6 pihole-FTL-full
endef

define Package/pihole-FTL-dhcpv6
$(call Package/pihole-FTL/Default)
  TITLE += (with DHCPv6 support)
  DEPENDS+=@IPV6
  VARIANT:=dhcpv6
  CONFLICTS+=pihole-FTL-full
  PROVIDES+=pihole-FTL dnsmasq-dhcpv6
endef

define Package/pihole-FTL-full
$(call Package/pihole-FTL/Default)
  TITLE += (with DNSSEC, DHCPv6, Auth DNS, IPset, Conntrack, NO_ID enabled by default)
  DEPENDS+=+PACKAGE_pihole-FTL_full_dnssec:libnettle \
	+PACKAGE_pihole-FTL_full_ipset:kmod-ipt-ipset \
	+PACKAGE_pihole-FTL_full_conntrack:libnetfilter-conntrack
  VARIANT:=full
  PROVIDES+=pihole-FTL pihole-FTL-dhcpv6 dnsmasq-dhcpv6 dnsmasq-full
endef

define Package/pihole-FTL/description
  FTLDNS™ (pihole-FTL) provides an interactive API and also generates statistics for Pi-hole®'s Web interface.
endef

define Package/pihole-FTL-dhcpv6/description
$(call Package/pihole-FTL/description)

This is a variant with DHCPv6 support
endef

define Package/pihole-FTL-full/description
$(call Package/pihole-FTL/description)

This is a fully configurable variant with DHCPv4, DHCPv6, DNSSEC, Authoritative DNS
and IPset, Conntrack support & NO_ID enabled by default.
endef

define Package/pihole-FTL/conffiles
/etc/config/dhcp
/etc/dnsmasq.conf
/etc/dnsmasq.d
/etc/pihole
endef

define Package/pihole-FTL-full/config
	if PACKAGE_pihole-FTL-full
	config PACKAGE_pihole-FTL_full_dhcp
		bool "Build with DHCP support."
		default y
	config PACKAGE_pihole-FTL_full_dhcpv6
		bool "Build with DHCPv6 support."
		depends on IPV6 && PACKAGE_pihole-FTL_full_dhcp
		default y
	config PACKAGE_pihole-FTL_full_dnssec
		bool "Build with DNSSEC support."
		default y
	config PACKAGE_pihole-FTL_full_auth
		bool "Build with the facility to act as an authoritative DNS server."
		default y
	config PACKAGE_pihole-FTL_full_ipset
		bool "Build with IPset support."
		default y
	config PACKAGE_pihole-FTL_full_conntrack
		bool "Build with Conntrack support."
		default y
	config PACKAGE_pihole-FTL_full_noid
		bool "Build with NO_ID. (hide *.bind pseudo domain)"
		default y
	config PACKAGE_pihole-FTL_full_broken_rtc
		bool "Build with HAVE_BROKEN_RTC."
		default n
	endif
endef

Package/pihole-FTL-dhcpv6/conffiles = $(Package/pihole-FTL/conffiles)
Package/pihole-FTL-full/conffiles = $(Package/pihole-FTL/conffiles)

TARGET_CFLAGS += -ffunction-sections -fdata-sections
TARGET_LDFLAGS += -Wl,--gc-sections

COPTS = $(if $(CONFIG_IPV6),,-DNO_IPV6)
LIBS = -pthread -lubox -lubus

ifeq ($(BUILD_VARIANT),nodhcpv6)
	COPTS += -DNO_DHCP6
endif

ifeq ($(BUILD_VARIANT),full)
	COPTS += $(if $(CONFIG_PACKAGE_pihole-FTL_$(BUILD_VARIANT)_dhcp),,-DNO_DHCP) \
		$(if $(CONFIG_PACKAGE_pihole-FTL_$(BUILD_VARIANT)_dhcpv6),,-DNO_DHCP6) \
		$(if $(CONFIG_PACKAGE_pihole-FTL_$(BUILD_VARIANT)_dnssec),-DHAVE_DNSSEC) \
		$(if $(CONFIG_PACKAGE_pihole-FTL_$(BUILD_VARIANT)_auth),,-DNO_AUTH) \
		$(if $(CONFIG_PACKAGE_pihole-FTL_$(BUILD_VARIANT)_ipset),,-DNO_IPSET) \
		$(if $(CONFIG_PACKAGE_pihole-FTL_$(BUILD_VARIANT)_conntrack),-DHAVE_CONNTRACK,) \
		$(if $(CONFIG_PACKAGE_pihole-FTL_$(BUILD_VARIANT)_noid),-DNO_ID,) \
		$(if $(CONFIG_PACKAGE_pihole-FTL_$(BUILD_VARIANT)_broken_rtc),-DHAVE_BROKEN_RTC)
	COPTS += $(if $(CONFIG_LIBNETTLE_MINI),-DNO_GMP,)
	LIBS += $(if $(CONFIG_PACKAGE_pihole-FTL_$(BUILD_VARIANT)_dnssec), \
		-lnettle $(if $(CONFIG_LIBNETTLE_MINI),,-lgmp) -lhogweed) \
		$(if $(CONFIG_PACKAGE_pihole-FTL_$(BUILD_VARIANT)_conntrack),-lnetfilter_conntrack,)
else
	COPTS += -DNO_AUTH -DNO_IPSET -DNO_ID
endif

MAKE_FLAGS := \
	$(TARGET_CONFIGURE_OPTS) \
	CCFLAGS="$(TARGET_CFLAGS) $(TARGET_CPPFLAGS)" \
	LDFLAGS="$(TARGET_LDFLAGS)" \
	DNSMASQOPTS="$(COPTS)" \
	LIBS="$(LIBS)" \
	PREFIX="/usr" \
	GIT_BRANCH="" \
	GIT_HASH="$(PKG_SOURCE_VERSION)" \
	GIT_VERSION="v$(PKG_VERSION)-openwrt" \
	GIT_DATE="$(shell date)" \
	GIT_TAG="v$(PKG_VERSION)" \
	SETCAP="0"

define Package/pihole-FTL/install
	$(INSTALL_DIR) $(1)/usr/bin
	$(CP) $(PKG_INSTALL_DIR)/usr/bin/pihole-FTL $(1)/usr/bin/
	$(INSTALL_DIR) $(1)/usr/sbin
	ln -s ../bin/pihole-FTL $(1)/usr/sbin/dnsmasq
	$(INSTALL_DIR) $(1)/etc/config
	$(INSTALL_DATA) ./files/dhcp.conf $(1)/etc/config/dhcp
	$(INSTALL_DATA) ./files/dnsmasq.conf $(1)/etc/dnsmasq.conf
	$(INSTALL_DIR) $(1)/etc/dnsmasq.d
	$(INSTALL_DATA) ./files/01-pihole.conf $(1)/etc/dnsmasq.d/
	$(INSTALL_DIR) $(1)/etc/pihole
	$(INSTALL_DATA) ./files/adlists.list $(1)/etc/pihole/
	$(INSTALL_DATA) ./files/pihole-FTL.conf $(1)/etc/pihole/
	$(INSTALL_DATA) ./files/setupVars.conf $(1)/etc/pihole/
	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) ./files/dnsmasq.init $(1)/etc/init.d/dnsmasq
	$(INSTALL_BIN) ./files/pihole-FTL.init $(1)/etc/init.d/pihole-FTL
	$(INSTALL_DIR) $(1)/etc/hotplug.d/dhcp
	$(INSTALL_DIR) $(1)/etc/hotplug.d/neigh
	$(INSTALL_DIR) $(1)/etc/hotplug.d/ntp
	$(INSTALL_DIR) $(1)/etc/hotplug.d/tftp
	$(INSTALL_DATA) ./files/piholesec.hotplug $(1)/etc/hotplug.d/ntp/25-piholesec
	$(INSTALL_DIR) $(1)/usr/share/pihole
	$(INSTALL_DATA) ./files/rfc6761.conf $(1)/usr/share/pihole/
	$(INSTALL_DIR) $(1)/usr/lib/pihole
	$(INSTALL_BIN) ./files/dhcp-script.sh $(1)/usr/lib/pihole/dhcp-script.sh
	$(INSTALL_DIR) $(1)/usr/share/acl.d
	$(INSTALL_DATA) ./files/pihole_acl.json $(1)/usr/share/acl.d/
endef

Package/pihole-FTL-dhcpv6/install = $(Package/pihole-FTL/install)

define Package/pihole-FTL-full/install
$(call Package/pihole-FTL/install,$(1))
ifneq ($(CONFIG_PACKAGE_pihole-FTL_full_dnssec),)
	$(INSTALL_DIR) $(1)/usr/share/pihole
	$(INSTALL_DATA) ./files/trust-anchors.conf $(1)/usr/share/pihole
endif
endef

$(eval $(call BuildPackage,pihole-FTL))
$(eval $(call BuildPackage,pihole-FTL-dhcpv6))
$(eval $(call BuildPackage,pihole-FTL-full))
