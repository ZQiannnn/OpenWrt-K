include $(TOPDIR)/rules.mk
include $(INCLUDE_DIR)/kernel.mk

PKG_NAME:=nf_deaf
PKG_VERSION:=1.0
PKG_RELEASE:=1

PKG_BUILD_DIR:=$(KERNEL_BUILD_DIR)/$(PKG_NAME)

include $(INCLUDE_DIR)/package.mk

define KernelPackage/nf_deaf
  SUBMENU:=Netfilter Extensions
  TITLE:=Netfilter deaf module for network testing
  DEPENDS:=+kmod-nf-conntrack
  FILES:=$(PKG_BUILD_DIR)/nf_deaf.ko
  AUTOLOAD:=$(call AutoProbe,nf_deaf)
  KCONFIG:=
endef

define KernelPackage/nf_deaf/description
  A kernel module for Linux routers that sends crafted packets after TCP handshake success.
  This module intercepts TCP packets marked with a specific magic number and generates
  crafted response packets for network testing purposes.

  Features:
  - Supports both IPv4 and IPv6
  - Configurable TTL, checksum corruption, sequence number manipulation
  - Delay and repeat functionality
  - Debugfs interface for runtime configuration

  WARNING: This module is for testing purposes only. Use responsibly and in compliance
  with applicable laws and regulations.
endef

define Build/Prepare
	mkdir -p $(PKG_BUILD_DIR)
	$(CP) ./src/* $(PKG_BUILD_DIR)/
endef

define Build/Compile
	+$(MAKE) $(PKG_JOBS) -C "$(LINUX_DIR)" \
		$(KERNEL_MAKE_FLAGS) \
		M="$(PKG_BUILD_DIR)" \
		modules
endef

define KernelPackage/nf_deaf/install
	$(INSTALL_DIR) $(1)/etc/modules.d
	echo 'nf_deaf' > $(1)/etc/modules.d/90-nf_deaf
endef

$(eval $(call KernelPackage,nf_deaf))
