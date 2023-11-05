all: build

########################################################################
##                             PROTOC                                 ##
########################################################################

# Only set PROTOC_VER if it has an empty value.
ifeq (,$(strip $(PROTOC_VER)))
PROTOC_VER := 24.0
endif

PROTOC_OS := $(shell uname -s)
ifeq (Darwin,$(PROTOC_OS))
PROTOC_OS := osx
endif

PROTOC_ARCH := $(shell uname -m)
ifeq (i386,$(PROTOC_ARCH))
PROTOC_ARCH := x86_32
endif

PROTOC := ./protoc
PROTOC_ZIP := protoc-$(PROTOC_VER)-$(PROTOC_OS)-$(PROTOC_ARCH).zip
PROTOC_URL := https://github.com/protocolbuffers/protobuf/releases/download/v$(PROTOC_VER)/$(PROTOC_ZIP)
PROTOC_TMP_DIR := .protoc
PROTOC_TMP_BIN := $(PROTOC_TMP_DIR)/bin/protoc

$(PROTOC):
	-mkdir -p "$(PROTOC_TMP_DIR)" && \
	  curl -L $(PROTOC_URL) -o "$(PROTOC_TMP_DIR)/$(PROTOC_ZIP)" && \
	  unzip "$(PROTOC_TMP_DIR)/$(PROTOC_ZIP)" -d "$(PROTOC_TMP_DIR)" && \
	  chmod 0755 "$(PROTOC_TMP_BIN)" && \
	  cp -f "$(PROTOC_TMP_BIN)" "$@"
	stat "$@" > /dev/null 2>&1

########################################################################
##                              BUILD                                 ##
########################################################################
CSI_PROTO := ../../csi.proto
CSI_PKG_ROOT := lib/cxx/$(PROTOC_TMP_DIR)
CSI_PKG_SUB := $(shell cat $(CSI_PROTO) | sed -n -e 's/^package.\([^;]*\).v[0-9]\+;$$/\1/p'|tr '.' '/')
CSI_BUILD := $(CSI_PKG_SUB)/.build
CSI_CXX := $(CSI_PKG_SUB)/csi.pb.cc
CSI_CXX_H := $(CSI_PKG_SUB)/csi.pb.h 
CSI_CXX_TMP := $(CSI_BUILD)/$(CSI_PKG_ROOT)/csi.pb.h $(CSI_BUILD)/$(CSI_PKG_ROOT)/csi.pb.cc

# This recipe generates the C++ language bindings to a temp area.
$(CSI_CXX_TMP): HERE := $(shell pwd)
$(CSI_CXX_TMP): CSI_ROOT := $(realpath $(HERE)/../../)
$(CSI_CXX_TMP): CXX_OUT := $(HERE)/$(CSI_BUILD)
$(CSI_CXX_TMP): INCLUDE := -I$(CSI_ROOT) -I$(HERE)/$(PROTOC_TMP_DIR)/include
$(CSI_CXX_TMP): $(CSI_PROTO) | $(PROTOC)
	@mkdir -p "$(CXX_OUT)"
	(cd "$(HERE)/$(PROTOC_TMP_DIR)" && \
		$(HERE)/$(PROTOC) $(INCLUDE) --cpp_out="$(CXX_OUT)" "$(CSI_PKG_ROOT)/$(<F)")

# The temp language bindings are compared to the ones that are
# versioned. If they are different then it means the language
# bindings were not updated prior to being committed.
$(CSI_CXX_H): $(CSI_BUILD)/$(CSI_PKG_ROOT)/csi.pb.h | $(CSI_CXX_TMP)
ifeq (true,$(GITHUB_ACTIONS))
	diff "$@" "$?"
else
	@mkdir -p "$(@D)"
	diff "$@" "$?" > /dev/null 2>&1 || cp -f "$?" "$@"
endif

$(CSI_CXX): $(CSI_BUILD)/$(CSI_PKG_ROOT)/csi.pb.cc | $(CSI_CXX_TMP)
ifeq (true,$(GITHUB_ACTIONS))
	diff "$@" "$?"
else
	@mkdir -p "$(@D)"
	diff "$@" "$?" > /dev/null 2>&1 || cp -f "$?" "$@"
endif

build: $(CSI_CXX_H) $(CSI_CXX)
	@echo "cxx bindings & validation"

clean:
	rm -rf "$(CSI_BUILD)" $(CSI_PKG_ROOT)/csi.proto

clobber: clean
	rm -rf "$(PROTOC)" "$(CSI_PKG_SUB)"

.PHONY: clean clobber
