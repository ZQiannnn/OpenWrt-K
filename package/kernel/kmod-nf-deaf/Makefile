# OpenWrt package Makefile for kmod-nf-deaf
include $(TOPDIR)/rules.mk
include $(INCLUDE_DIR)/kernel.mk

PKG_NAME:=kmod-nf-deaf
PKG_RELEASE:=1

include $(INCLUDE_DIR)/package.mk

define KernelPackage/nf-deaf
  SUBMENU:=Netfilter Extensions
  TITLE:=Netfilter DEAF module
  FILES:=$(PKG_BUILD_DIR)/nf_deaf.ko
  AUTOLOAD:=$(call AutoLoad,80,nf_deaf)
  DEPENDS:=+kmod-nf-conntrack +kmod-ipt-core
endef

define KernelPackage/nf-deaf/description
  This package contains the nf_deaf kernel module for netfilter packet manipulation.
  It provides custom packet processing capabilities through netfilter hooks.
endef

define Build/Prepare
	mkdir -p $(PKG_BUILD_DIR)
	$(CP) ./src/* $(PKG_BUILD_DIR)/
endef

define Build/Compile
	$(MAKE) -C "$(LINUX_DIR)" \
		CROSS_COMPILE="$(TARGET_CROSS)" \
		ARCH="$(LINUX_KARCH)" \
		SUBDIRS="$(PKG_BUILD_DIR)" \
		EXTRA_CFLAGS="$(BUILDFLAGS)" \
		modules
endef

$(eval $(call KernelPackage,nf-deaf))
