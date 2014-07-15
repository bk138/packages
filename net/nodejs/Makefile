#
# Copyright (C) 2006-2011 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#

include $(TOPDIR)/rules.mk

PKG_NAME:=node
PKG_VERSION:=v0.10.17
PKG_RELEASE:=2

PKG_SOURCE:=node-$(PKG_VERSION).tar.gz
PKG_SOURCE_URL:=http://nodejs.org/dist/${PKG_VERSION}
PKG_MD5SUM:=a2b05af77e8e3ef3b4c40a68372429f1
GIT_SOURCE:=https://github.com/brimstone/v8m-rb
#GIT_SOURCE:=/tmp/v8m-rb

GYP_DEFINES:=v8_use_mips_abi_hardfloat=false v8_can_use_fpu_instructions=false
LIBS:=-I$(TOOLCHAIN_DIR)/mips-openwrt-linux-uclibc/include/c++/4.6.3/ -I$(TOOLCHAIN_DIR)/mips-openwrt-linux-uclibc/include/c++/4.6.3/mips-openwrt-linux-uclibc/

include $(INCLUDE_DIR)/package.mk

define Package/node
  DEPENDS:=+libpthread +librt +uclibcxx $(if $(CONFIG_PACKAGE_libopenssl), +libcryptoxx +libopenssl)
  SECTION:=lang
  CATEGORY:=Languages
  TITLE:=Node.js is a platform built on Chrome's JavaScript runtime
  URL:=http://nodejs.org/
  MAINTAINER:=Christian Beier <cb@shoutrlabs.com>
endef


define Package/node/description
Node.js is a platform built on Chrome's JavaScript runtime for easily building fast, scalable network applications. Node.js uses an event-driven, non-blocking I/O model that makes it lightweight and efficient, perfect for data-intensive real-time applications that run across distributed devices.
endef


define Package/npm
  SECTION:=lang
  CATEGORY:=Languages
  TITLE:=Nodejs Package Manager
  URL:=http://nodejs.org/
  DEPENDS:= +node
endef
 
define Package/npm/description
 Nodejs Package Manager
endef




define Build/Prepare
	$(call Build/Prepare/Default)
	$(CP) node.patch $(PKG_BUILD_DIR)/
	(cd $(PKG_BUILD_DIR); \
	rm -rf deps/v8; \
	git clone $(GIT_SOURCE) deps/v8; \
	patch -p1 < node.patch; \
	);
endef

define Build/Configure
	(cd $(PKG_BUILD_DIR); \
	export LIBS="$(LIBS)"; \
	export CFLAGS="$(TARGET_CFLAGS) $(LIBS)"; \
	export CXXFLAGS="$(TARGET_CXXFLAGS) $(LIBS)"; \
    export GYPFLAGS="$(GYPFLAGS)"; \
	$(PYTHON) ./configure \
		--dest-cpu=mips \
		--dest-os=linux \
		--without-snapshot \
		$(if $(CONFIG_PACKAGE_libopenssl),--shared-openssl) \
		--with-arm-float-abi=soft; \
	);
endef

define Build/Compile
	$(MAKE) $(PKG_JOBS) -C $(PKG_BUILD_DIR) GYP_DEFINES="$(GYP_DEFINES)" CC="$(TARGET_CC)" CXX="$(TARGET_CXX)" CFLAGS="$(TARGET_CFLAGS) $(LIBS)" CXXFLAGS="$(TARGET_CXXFLAGS) $(LIBS) -nostdinc++" LDFLAGS="$(TARGET_LDFLAGS) -nodefaultlibs -luClibc++ -lc -lgcc -lgcc_s -lpthread" || touch $(PKG_BUILD_DIR)/deps/v8/build/common.gypi
	$(MAKE) $(PKG_JOBS) -C $(PKG_BUILD_DIR) GYP_DEFINES="$(GYP_DEFINES)" CC="$(TARGET_CC)" CXX="$(TARGET_CXX)" CFLAGS="$(TARGET_CFLAGS) $(LIBS)" CXXFLAGS="$(TARGET_CXXFLAGS) $(LIBS) -nostdinc++" LDFLAGS="$(TARGET_LDFLAGS) -nodefaultlibs -luClibc++ -lc -lgcc -lgcc_s -lpthread"
endef


define Package/node/install
	mkdir -p $(1)/usr/bin
	$(CP) $(PKG_BUILD_DIR)/out/Release/node $(1)/usr/bin/
	ln -s /usr/bin/node $(1)/usr/bin/nodejs
endef

define Package/npm/install
	mkdir -p $(1)/usr/bin
	mkdir -p $(1)/usr/share/npm/bin
	$(CP) $(PKG_BUILD_DIR)/deps/npm/bin/npm-cli.js $(1)/usr/share/npm/bin/
	$(CP) $(PKG_BUILD_DIR)/deps/npm/bin/read-package-json.js $(1)/usr/share/npm/bin/
	$(CP) $(PKG_BUILD_DIR)/deps/npm/lib $(1)/usr/share/npm/
	$(CP) $(PKG_BUILD_DIR)/deps/npm/node_modules $(1)/usr/share/npm/
	ln -s /usr/share/npm/bin/npm-cli.js $(1)/usr/bin/npm
endef

 

$(eval $(call BuildPackage,node))
$(eval $(call BuildPackage,npm))
