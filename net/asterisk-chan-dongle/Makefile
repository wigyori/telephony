#
# Copyright (C) 2017 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#

include $(TOPDIR)/rules.mk

PKG_NAME:=asterisk-chan-dongle
PKG_VERSION:=1.1-20170913
PKG_RELEASE:=2

PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.xz
PKG_SOURCE_URL:=https://github.com/wdoekes/asterisk-chan-dongle.git
PKG_SOURCE_SUBDIR:=$(PKG_NAME)-$(PKG_VERSION)
PKG_SOURCE_VERSION:=4ef5ad7eea7245a031101875be08b924aa1e151b
PKG_SOURCE_PROTO:=git

PKG_FIXUP:=autoreconf

PKG_LICENSE:=GPL-2.0
PKG_LICENSE_FILES:=COPYRIGHT.txt LICENSE.txt
PKG_MAINTAINER:=Jiri Slachta <jiri@slachta.eu>

PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)-$(BUILD_VARIANT)/$(PKG_NAME)-$(PKG_VERSION)

include $(INCLUDE_DIR)/package.mk

define Package/asterisk-chan-dongle/Default
  SUBMENU:=Telephony
  SECTION:=net
  CATEGORY:=Network
  URL:=https://github.com/wdoekes/asterisk-chan-dongle
  DEPENDS:=+USE_UCLIBC:libiconv-full +kmod-usb-acm +kmod-usb-serial +kmod-usb-serial-option +libusb-1.0 +usb-modeswitch
  TITLE:=Huawei UMTS 3G dongle support
endef

define Package/asterisk13-chan-dongle
$(call Package/asterisk-chan-dongle/Default)
  DEPENDS+=asterisk13
  VARIANT:=asterisk13
endef

define Package/description/Default
 Asterisk channel driver for Huawei UMTS 3G dongle.
endef

Package/asterisk13-chan-dongle/description = $(Package/description/Default)

ifeq ($(BUILD_VARIANT),asterisk13)
  CHAN_DONGLE_AST_HEADERS:=$(STAGING_DIR)/usr/include/asterisk-13/include
  CONFIGURE_ARGS+= \
	  --with-astversion=13
endif

CONFIGURE_ARGS+= \
	--with-asterisk=$(CHAN_DONGLE_AST_HEADERS)

TARGET_CFLAGS+= \
	-I$(CHAN_DONGLE_AST_HEADERS)

# musl and glibc include their own iconv, but uclibc does not
ifneq ($(CONFIG_USE_UCLIBC),)
TARGET_CPPFLAGS+= \
	-I$(STAGING_DIR)/usr/lib/libiconv-full/include
endif

# -DAST_MODULE_SELF_SYM=__internal_chan_dongle_self to fix Asterisk 14
# compile issues
CHAN_DONGLE_EXTRA_CFLAGS:= \
	-Wno-old-style-declaration \
	-I$(PKG_BUILD_DIR) \
	-DAST_MODULE_SELF_SYM=__internal_chan_dongle_self \
	$(TARGET_CPPFLAGS) \
	-D_GNU_SOURCE \
	-DHAVE_CONFIG_H \
	$(FPIC)

MAKE_ARGS:= \
	CC="$(TARGET_CC)" \
	LD="$(TARGET_CC)" \
	CFLAGS="$(TARGET_CFLAGS) $(CHAN_DONGLE_EXTRA_CFLAGS)" \
	LDFLAGS="$(TARGET_LDFLAGS) $(if $(CONFIG_USE_UCLIBC),-L$(STAGING_DIR)/usr/lib/libiconv-full/lib -liconv)"

# $CHAN_DONGLE_ICONV_INC used by 200-fix-iconv-detection.patch
CONFIGURE_VARS += \
	CHAN_DONGLE_ICONV_INC="$(TOOLCHAIN_DIR)/include $(STAGING_DIR)/usr/lib/libiconv-full/include" \
	ac_cv_type_size_t=yes \
	ac_cv_type_ssize_t=yes

define Build/Compile
	$(MAKE) $(PKG_JOBS) -C "$(PKG_BUILD_DIR)" $(MAKE_ARGS)
endef

define Package/conffiles/Default
/etc/asterisk/dongle.conf
endef

Package/asterisk13-chan-dongle/conffiles = $(Package/conffiles/Default)

define Package/Install/Default
	$(INSTALL_DIR) $(1)/etc/asterisk
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/etc/dongle.conf $(1)/etc/asterisk
	$(INSTALL_DIR) $(1)/usr/lib/asterisk/modules
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/chan_dongle.so $(1)/usr/lib/asterisk/modules
endef

Package/asterisk13-chan-dongle/install = $(Package/Install/Default)

$(eval $(call BuildPackage,asterisk13-chan-dongle))
