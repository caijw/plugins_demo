# linux app 的 makefile

include app_configuration.mk

# Default build type.
BUILD=debug

# CURDIR : appRoot/linux/

# appRoot/linux/flutter
# 命令行工具自动生成的
FLUTTER_MANAGED_DIR=flutter
# appRoot/linux/flutter/ephemeral 存放了从flutter engine 哪里 
# copy 过来的文件，一个临时的目录，appRoot/linux 下面 make clean 会删掉这个目录
FLUTTER_EPHEMERAL_DIR=$(FLUTTER_MANAGED_DIR)/ephemeral

# Configuration provided via flutter tool.
# FLUTTER_ROOT flutter 命令行工具位置 FLUTTER_TARGET marin.dart 位置 PROJECT_DIR 项目目录位置 等配置，
FLUTTER_CONFIG_FILE=$(FLUTTER_EPHEMERAL_DIR)/generated_config.mk

include $(FLUTTER_CONFIG_FILE)

# Dependency locations
# app 根目录 appRoot
FLUTTER_APP_DIR=$(CURDIR)/..

# appRoot/build
FLUTTER_APP_BUILD_DIR=$(FLUTTER_APP_DIR)/build
# appRoot/build/linux
OUT_DIR=$(FLUTTER_APP_BUILD_DIR)/linux
# appRoot/build/linux/obj/debug
OBJ_DIR=$(OUT_DIR)/obj/$(BUILD)

# Libraries
# flutter engine 动态链接库名字
FLUTTER_LIB_NAME=flutter_linux_glfw
# flutter engine 动态链接库位置
FLUTTER_LIB=$(FLUTTER_EPHEMERAL_DIR)/lib$(FLUTTER_LIB_NAME).so

# Tools
# flutter 命令行工具位置
FLUTTER_BIN=$(FLUTTER_ROOT)/bin/flutter
# linux build sh 脚本
LINUX_BUILD=$(FLUTTER_ROOT)/packages/flutter_tools/bin/tool_backend.sh

# Resources
# 资源
ICU_DATA_NAME=icudtl.dat
ICU_DATA_SOURCE=$(FLUTTER_EPHEMERAL_DIR)/$(ICU_DATA_NAME)
FLUTTER_ASSETS_NAME=flutter_assets
FLUTTER_ASSETS_SOURCE=$(FLUTTER_APP_BUILD_DIR)/$(FLUTTER_ASSETS_NAME)

# Bundle structure
# 打包目录相关
# appRoot/build/linux/debug
BUNDLE_OUT_DIR=$(OUT_DIR)/$(BUILD)
# appRoot/build/linux/debug/data
BUNDLE_DATA_DIR=$(BUNDLE_OUT_DIR)/data
# appRoot/build/linux/debug/lib
BUNDLE_LIB_DIR=$(BUNDLE_OUT_DIR)/lib
# app 二进制输出位置 appRoot/build/linux/debug/appName
BIN_OUT=$(BUNDLE_OUT_DIR)/$(BINARY_NAME)

ICU_DATA_OUT=$(BUNDLE_DATA_DIR)/$(ICU_DATA_NAME)
# flutter engine 动态链接库输出位置
# appRoot/build/linux/debug/lib/libflutter_linux_glfw.so
FLUTTER_LIB_OUT=$(BUNDLE_LIB_DIR)/$(notdir $(FLUTTER_LIB))
# 所有的动态库文件
# FLUTTER_LIB_OUT flutter engine 动态库
# EXTRA_BUNDLED_LIBRARIES 插件动态库等，appRoot/linux/flutter/ephemeral/.plugin_symlinks下面存放了所有插件的软连接，插件编译后的so会复制到 appRoot/build/linux/lib 下面
# appRoot/build/linux/debug/lib/$(notdir $(lib)) $(lib) 这个还没找到是哪里定义的
ALL_LIBS_OUT=$(FLUTTER_LIB_OUT) \
	$(foreach lib,$(EXTRA_BUNDLED_LIBRARIES),$(BUNDLE_LIB_DIR)/$(notdir $(lib)))

# Add relevant code from the wrapper library, which is intended to be statically
# built into the client.
# Use abspath for the wrapper root, which can contain relative paths; the
# intermediate build files will be based on the source path, which will cause
# issues if they start with one or more '../'s.
WRAPPER_ROOT=$(abspath $(FLUTTER_EPHEMERAL_DIR)/cpp_client_wrapper_glfw)
WRAPPER_SOURCES= \
	$(WRAPPER_ROOT)/flutter_window_controller.cc \
	$(WRAPPER_ROOT)/plugin_registrar.cc \
	$(WRAPPER_ROOT)/engine_method_result.cc

# Use abspath for extra sources, which may also contain relative paths (see
# note above about WRAPPER_ROOT).
# app 的 main 文件和一些需要编译的源文件
SOURCES=main.cc window_configuration.cc \
	flutter/generated_plugin_registrant.cc \
	$(WRAPPER_SOURCES) $(abspath $(EXTRA_SOURCES))

# Headers
WRAPPER_INCLUDE_DIR=$(WRAPPER_ROOT)/include
INCLUDE_DIRS=$(FLUTTER_EPHEMERAL_DIR) $(WRAPPER_INCLUDE_DIR)

# Build settings
ifneq ($(strip $(SYSTEM_LIBRARIES)),)
EXTRA_CPPFLAGS+=$(patsubst -I%,-isystem%,$(shell pkg-config --cflags $(SYSTEM_LIBRARIES)))
EXTRA_LDFLAGS+=$(shell pkg-config --libs $(SYSTEM_LIBRARIES))
endif
CXX=clang++
# CPPFLAGS
CPPFLAGS.release=-DNDEBUG
CPPFLAGS.profile=$(CPPFLAGS.release)
# CXXFLAGS
CXXFLAGS.release=-O2
CXXFLAGS.profile=$(CXXFLAGS.release)
CXXFLAGS=-std=c++14 -Wall -Werror $(CXXFLAGS.$(BUILD)) $(EXTRA_CXXFLAGS)
CPPFLAGS=$(patsubst %,-I%,$(INCLUDE_DIRS)) \
	$(CPPFLAGS.$(BUILD)) $(EXTRA_CPPFLAGS)

# 动态库相关编译配置
LDFLAGS=-L$(BUNDLE_LIB_DIR) \
	-l$(FLUTTER_LIB_NAME) \
	$(EXTRA_LDFLAGS) \
	-Wl,-rpath=\$$ORIGIN/lib


$(info [log] CPPFLAGS: ${CPPFLAGS})
$(info [log] CXXFLAGS: ${CXXFLAGS})
$(info [log] LDFLAGS: ${LDFLAGS})

# Intermediate files.
OBJ_FILES=$(SOURCES:%.cc=$(OBJ_DIR)/%.o)
DEPENDENCY_FILES=$(OBJ_FILES:%.o=%.d)

$(info [log] OBJ_FILES: ${OBJ_FILES})
$(info [log] DEPENDENCY_FILES: ${DEPENDENCY_FILES})

# Targets

.PHONY: all
all: $(BIN_OUT) bundle

# Add the plugin targets, and their associated settings.
# 引入 plugins 插件的 makefile 配置
# 这份配置是命令行工具自动生成的，包含 plugin 的编译配置
include $(FLUTTER_MANAGED_DIR)/generated_plugins.mk
# PLUGIN_LIBRARIES: appRoot/build/linux/lib$(plugin).so 插件的so
EXTRA_BUNDLED_LIBRARIES+=$(PLUGIN_LIBRARIES)
# 插件动态库 LDFLAGS
EXTRA_LDFLAGS+=$(PLUGIN_LDFLAGS)
# 插件动态库 CPPFLAGS
EXTRA_CPPFLAGS+=$(PLUGIN_CPPFLAGS)

$(info [log] EXTRA_BUNDLED_LIBRARIES: $(EXTRA_BUNDLED_LIBRARIES))
$(info [log] EXTRA_LDFLAGS $(EXTRA_LDFLAGS))
$(info [log] EXTRA_CPPFLAGS $(EXTRA_CPPFLAGS))

# This is a phony target because the flutter tool cannot describe
# its inputs and outputs yet.
.PHONY: sync
# 运行 linux 构建 shell 脚本
sync: $(FLUTTER_CONFIG_FILE)
	$(LINUX_BUILD) linux-x64 $(BUILD)

.PHONY: bundle
# 所有的资源文件、动态库准备完成
bundle: $(ICU_DATA_OUT) $(ALL_LIBS_OUT) bundleflutterassets
#  编译 app
$(BIN_OUT): $(OBJ_FILES) $(ALL_LIBS_OUT)
	mkdir -p $(@D)
	$(CXX) $(CXXFLAGS) $(CPPFLAGS) $(OBJ_FILES) $(LDFLAGS) -o $@

$(WRAPPER_SOURCES) $(FLUTTER_LIB) $(ICU_DATA_SOURCE) $(FLUTTER_ASSETS_SOURCE) \
	$(PLUGIN_TARGETS): | sync

# Plugin library bundling pattern.
# OUT_DIR 复制到 BUNDLE_LIB_DIR ，目前看就是把 OUT_DIR 下的 so 复制到 BUNDLE_LIB_DIR
$(BUNDLE_LIB_DIR)/%: $(OUT_DIR)/%
	mkdir -p $(BUNDLE_LIB_DIR)
	cp $< $@
# flutter engine 动态链接库输出到 build 目录下
$(FLUTTER_LIB_OUT): $(FLUTTER_LIB)
	mkdir -p $(@D)
	cp $< $@

$(ICU_DATA_OUT): $(ICU_DATA_SOURCE)
	mkdir -p $(@D)
	cp $< $@

# xxx.d 文件，避免头文件改动后没有触发编译更新
-include $(DEPENDENCY_FILES)
# 编译生成 .o 文件，使用了 -MMD 参数生成 .d 头文件依赖描述
$(OBJ_DIR)/%.o : %.cc | sync
	mkdir -p $(@D)
	$(CXX) $(CXXFLAGS) $(CPPFLAGS) -MMD -c $< -o $@

# Fully re-copy the assets directory on each build to avoid having to keep a
# comprehensive list of all asset files here, which would be fragile to changes
# in other files (e.g., adding a new font to pubspec.yaml).
.PHONY: bundleflutterassets

bundleflutterassets: $(FLUTTER_ASSETS_SOURCE)
	mkdir -p $(BUNDLE_DATA_DIR)
	rsync -rpu --delete $(FLUTTER_ASSETS_SOURCE) $(BUNDLE_DATA_DIR)

.PHONY: clean

clean:
	rm -rf $(OUT_DIR); \
	cd $(FLUTTER_APP_DIR); \
	$(FLUTTER_BIN) clean
