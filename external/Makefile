
LOCAL_PATH := $(shell pwd)

PROJECT_ROOT = $(LOCAL_PATH)/..
EXTERNAL_ROOT = $(PROJECT_ROOT)/external

# Android now has 64-bit and 32-bit versions of the NDK for GNU/Linux.  We
# assume that the build platform uses the appropriate version, otherwise the
# user building this will have to manually set NDK_PROCESSOR or NDK_TOOLCHAIN.
CPU := $(shell uname -m)
ifeq ($(CPU),x86_64)
 NDK_PROCESSOR=x86_64
else
 NDK_PROCESSOR=x86
endif

# Android NDK setup
NDK_BASE ?= /opt/android-ndk
NDK_ABI=arm
# NDK platform level, aka APP_PLATFORM, is equivalent to minSdkVersion
NDK_PLATFORM_LEVEL ?= \
	$(shell sed -n 's,.*android:minSdkVersion="\([0-9][0-9]*\)".*,\1,p' $(PROJECT_ROOT)/AndroidManifest.xml)
NDK_TOOLCHAIN_VERSION = \
	$(shell sed -n 's,.*NDK_TOOLCHAIN_VERSION=\(.*\),\1,p' $(PROJECT_ROOT)/jni/Application.mk)
NDK_SYSROOT=$(NDK_BASE)/platforms/android-$(NDK_PLATFORM_LEVEL)/arch-$(NDK_ABI)
NDK_UNAME:=$(shell uname -s | tr '[A-Z]' '[a-z]')
ifeq ($(NDK_ABI),x86)
 HOST = i686-linux-android
 NDK_TOOLCHAIN = $(NDK_ABI)-$(NDK_TOOLCHAIN_VERSION)
else
 HOST = $(NDK_ABI)-linux-androideabi
 NDK_TOOLCHAIN = $(HOST)-$(NDK_TOOLCHAIN_VERSION)
endif
NDK_TOOLCHAIN_BASE=$(NDK_BASE)/toolchains/$(NDK_TOOLCHAIN)/prebuilt/$(NDK_UNAME)-$(NDK_PROCESSOR)


all: openssl/libs/armeabi/libcrypto.so sqlcipher/sqlite3.c

# TODO include this Android.mk to integrate this into the build
openssl/libs/armeabi/libcrypto.so:
	cd openssl && ndk-build -j4

openssl-clean:
	-cd openssl && ndk-build clean


# This target only generates the sqlite3.c source file, which is later built
# by jni/Android.mk.  Therefore, we don't need it to have any of the actual
# build flags.  We also rely on the build host system to provide the
# dependencies, like openssl, rather than point it to the included sources
# which are built for Android.
sqlcipher/sqlite3.c:
	cd sqlcipher && ./configure --enable-tempstore=yes \
		CFLAGS="-DSQL_HAS_CODEC"
	make -C sqlcipher sqlite3.c

sqlcipher-clean:
	-make -C sqlcipher clean
	-rm sqlcipher/sqlite3.c


clean: openssl-clean sqlcipher-clean

.PHONY: clean openssl-clean sqlcipher-clean
