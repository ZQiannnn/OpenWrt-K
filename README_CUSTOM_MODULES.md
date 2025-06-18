# OpenWrt-K 自定义内核模块集成说明

## 概述

本文档说明了如何将自定义内核模块（如 `nf_deaf`）集成到 OpenWrt-K 构建系统中。

## 目录结构

```
OpenWrt-K/
├── package/
│   └── kernel/
│       └── kmod-nf-deaf/
│           ├── Makefile          # OpenWrt 包定义
│           └── src/
│               ├── Makefile      # 内核模块编译规则
│               └── nf_deaf.c     # 源代码
├── config/
│   ├── x86_64/
│   │   └── kmod.config          # 添加了 CONFIG_PACKAGE_kmod-nf-deaf=y
│   └── rpi4b/
│       └── kmod.config          # 添加了 CONFIG_PACKAGE_kmod-nf-deaf=y
└── my_source/                   # 原始源码（保留）
    ├── Makefile
    └── nf_deaf.c
```

## GitHub Actions 构建流程

### 1. 准备阶段 (prepare action)
- **位置**: `.github/action/prepare/action.yml`
- **操作**: `cp -RT $GITHUB_WORKSPACE /opt/OpenWrt-K`
- **作用**: 将整个项目（包括 `package/` 目录）复制到构建环境

### 2. 源码准备 (prepare.py)
- **函数**: `prepare_cfg()` 在 `build_helper/prepare.py`
- **关键代码**:
  ```python
  # 复制本地自定义包
  logger.info("%s复制本地自定义包...", cfg_name)
  local_package_path = os.path.join(paths.openwrt_k, "package")
  if os.path.exists(local_package_path):
      for item in os.listdir(local_package_path):
          src_path = os.path.join(local_package_path, item)
          if os.path.isdir(src_path):
              dst_path = os.path.join(openwrt.path, "package", item)
              logger.debug("复制本地包目录 %s 到 %s", src_path, dst_path)
              if os.path.exists(dst_path):
                  shutil.rmtree(dst_path)
              shutil.copytree(src_path, dst_path, symlinks=True)
  ```

### 3. 配置应用
- **配置文件**: `config/{platform}/kmod.config`
- **内容**: `CONFIG_PACKAGE_kmod-nf-deaf=y`
- **作用**: 启用自定义内核模块的编译

### 4. 编译过程
- **base-builds**: 编译工具链和内核
- **build-ImageBuilder**: 编译内核模块，生成 `.ipk` 包
- **build-packages**: 编译用户空间软件包

## 包定义文件说明

### package/kernel/kmod-nf-deaf/Makefile
```makefile
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
```

### package/kernel/kmod-nf-deaf/src/Makefile
```makefile
obj-m += nf_deaf.o
```

## 添加新的自定义模块

1. **创建包目录**:
   ```bash
   mkdir -p package/kernel/kmod-your-module/src
   ```

2. **创建包 Makefile**:
   - 复制 `package/kernel/kmod-nf-deaf/Makefile`
   - 修改包名和相关配置

3. **添加源码**:
   - 将源码文件放入 `src/` 目录
   - 创建简单的 `src/Makefile`

4. **更新配置**:
   - 在 `config/{platform}/kmod.config` 中添加 `CONFIG_PACKAGE_kmod-your-module=y`

5. **提交更改**:
   - GitHub Actions 会自动检测更改并重新构建

## 注意事项

1. **依赖关系**: 确保在包定义中正确声明依赖关系
2. **命名规范**: 包名应遵循 `kmod-` 前缀的命名规范
3. **许可证**: 确保源码包含正确的许可证声明
4. **测试**: 建议在本地环境先测试模块编译

## 构建输出

编译完成后，内核模块将生成为：
- **IPK 包**: `bin/packages/*/kernel/kmod-nf-deaf_*.ipk`
- **安装**: 模块会自动在系统启动时加载（优先级 80）

## 故障排除

1. **编译失败**: 检查源码语法和依赖关系
2. **模块未加载**: 检查 AUTOLOAD 配置和依赖模块
3. **配置未生效**: 确认配置文件中的包名正确
