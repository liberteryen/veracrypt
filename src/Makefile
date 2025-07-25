#
# Derived from source code of TrueCrypt 7.1a, which is
# Copyright (c) 2008-2012 TrueCrypt Developers Association and which is governed
# by the TrueCrypt License 3.0.
#
# Modifications and additions to the original source code (contained in this file)
# and all other portions of this file are Copyright (c) 2025 AM Crypto
# and are governed by the Apache License 2.0 the full text of which is
# contained in the file License.txt included in VeraCrypt binary and source
# code distribution packages.
#

#------ Command line arguments ------
# DEBUG:		Disable optimizations and enable debugging checks
# DEBUGGER:		Enable debugging information for use by debuggers
# NOASM:		Exclude modules requiring assembler
# NOGUI:		Disable graphical user interface (build console-only application)
# NOSTRIP:		Do not strip release binary
# NOTEST:		Do not test release binary
# RESOURCEDIR:	Run-time resource directory
# VERBOSE:		Enable verbose messages
# WXSTATIC:		Use static wxWidgets library
# SSSE3:		Enable SSSE3 support in compiler
# SSE41:		Enable SSE4.1 support in compiler
# NOSSE2:		Disable SEE2 support in compiler
# WOLFCRYPT:	Build with wolfCrypt as crypto provider (see Crypto/wolfCrypt.md)
# WITHFUSET:	Build with FUSE-T support on macOS instead of MacFUSE

#------ Targets ------
# all
# clean
# wxbuild:		Configure and build wxWidgets - source code must be located at $(WX_ROOT)


#------ Build configuration ------

export APPNAME := veracrypt
export BASE_DIR := $(CURDIR)
export BUILD_INC := $(BASE_DIR)/Build/Include

export AR ?= ar
export CC ?= gcc
export CXX ?= g++
export AS := yasm
export RANLIB ?= ranlib

export CFLAGS := -Wall
export CXXFLAGS := -Wall -Wno-unused-parameter
C_CXX_FLAGS := -MMD -D_FILE_OFFSET_BITS=64 -D_LARGEFILE_SOURCE -D_LARGE_FILES -I$(BASE_DIR) -I$(BASE_DIR)/Crypto -DARGON2_NO_THREADS -I$(BASE_DIR)/Crypto/Argon2/include
export ASFLAGS := -D __GNUC__ -D __YASM__
export LFLAGS :=

export PKG_CONFIG ?= pkg-config
export PKG_CONFIG_PATH ?= /usr/local/lib/pkgconfig
export VC_FUSE_PACKAGE := fuse
export VC_OSX_FUSET ?= 0

export WX_CONFIG ?= wx-config
export WX_CONFIG_ARGS := --unicode
WX_CONFIGURE_FLAGS :=
export WXCONFIG_CFLAGS :=
export WXCONFIG_CXXFLAGS :=
WX_ROOT ?= $(BASE_DIR)/wxWidgets

export TC_BUILD_CONFIG := Release

ifeq "$(origin WITHFUSET)" "command line"
	ifneq "$(WITHFUSET)" "0"
		VC_OSX_FUSET := 1
	endif
endif

ifeq "$(origin DEBUG)" "command line"
	ifneq "$(DEBUG)" "0"
		TC_BUILD_CONFIG := Debug
	endif
endif

ifeq "$(origin NOGUI)" "command line"
	export TC_NO_GUI := 1
	C_CXX_FLAGS += -DTC_NO_GUI -DwxUSE_GUI=0
	WX_CONFIGURE_FLAGS += --disable-gui
endif

ifdef PKCS11_INC
	C_CXX_FLAGS += -I$(PKCS11_INC)
else
	C_CXX_FLAGS += -I$(CURDIR)/PKCS11
endif

ifeq "$(origin RESOURCEDIR)" "command line"
	C_CXX_FLAGS += -DTC_RESOURCE_DIR="$(RESOURCEDIR)"
endif

ifneq "$(origin VERBOSE)" "command line"
	MAKEFLAGS += -s
endif

ifeq "$(origin WXSTATIC)" "command line"
	export VC_WX_STATIC := 1
	WX_CONFIG = $(WX_BUILD_DIR)/wx-config
	WX_CONFIG_ARGS += --static
	ifneq "$(WXSTATIC)" "FULL"
		export VC_WX_MINIMAL := 1
	endif
endif

#------ Release configuration ------

ifeq "$(TC_BUILD_CONFIG)" "Release"

	C_CXX_FLAGS += -O2 -fno-strict-aliasing  # Do not enable strict aliasing
	export WX_BUILD_DIR ?= $(BASE_DIR)/wxrelease
	WX_CONFIGURE_FLAGS += --disable-debug_flag --disable-debug_gdb --disable-debug_info

else

#------ Debug configuration ------

	C_CXX_FLAGS += -DDEBUG
	CXXFLAGS += -fno-default-inline -Wno-unused-function -Wno-unused-variable
	export WX_BUILD_DIR ?= $(BASE_DIR)/wxdebug
	WX_CONFIGURE_FLAGS += --enable-debug_flag --disable-debug_gdb --disable-debug_info

endif


#------ Debugger configuration ------

ifeq "$(origin DEBUGGER)" "command line"

	C_CXX_FLAGS += -ggdb
	WX_CONFIGURE_FLAGS += --enable-debug_gdb --enable-debug_info

endif


#------ Platform configuration ------

export PLATFORM := "Unknown"
export PLATFORM_ARCH := "Unknown"
export PLATFORM_UNSUPPORTED := 0

export CPU_ARCH ?= unknown
export SIMD_SUPPORTED := 0
export DISABLE_AESNI ?= 0
export ENABLE_WOLFCRYPT ?= 0

export GCC_GTEQ_440 := 0
export GCC_GTEQ_430 := 0
export GCC_GTEQ_470 := 0
export GCC_GTEQ_500 := 0
export GTK_VERSION := 0

ARCH ?= $(shell uname -m)

ifneq (,$(filter i386 i486 i586 i686 x86,$(ARCH)))
	CPU_ARCH = x86
    ASFLAGS += -f elf32 -D __BITS__=32
else ifneq (,$(filter x86_64 x86-64 amd64 x64,$(ARCH)))
	CPU_ARCH = x64
    ASFLAGS += -f elf64 -D __BITS__=64
else ifneq (,$(filter armv7l,$(ARCH)))
	PLATFORM_ARCH := armv7
	CPU_ARCH = armv7
else ifneq (,$(filter aarch64 arm64 armv8l,$(ARCH)))
	PLATFORM_ARCH := arm64
	CPU_ARCH = arm64
endif

ifeq "$(origin NOASM)" "command line"
	CPU_ARCH = unknown
	C_CXX_FLAGS += -DCRYPTOPP_DISABLE_X86ASM
endif

ifeq "$(CPU_ARCH)" "x86"
	PLATFORM_ARCH := i386
	SIMD_SUPPORTED := 1
	C_CXX_FLAGS += -D TC_ARCH_X86
else ifeq "$(CPU_ARCH)" "x64"
	PLATFORM_ARCH := amd64
	SIMD_SUPPORTED := 1
	C_CXX_FLAGS += -D TC_ARCH_X64
endif

ifeq "$(origin NOSSE2)" "command line"
	SIMD_SUPPORTED := 0
endif

ifeq "$(origin NOAESNI)" "command line"
	DISABLE_AESNI := 1
endif

ifeq "$(origin WOLFCRYPT)" "command line"
	ENABLE_WOLFCRYPT := 1
	C_CXX_FLAGS += -DWOLFCRYPT_BACKEND
	export LIBS += -lwolfssl
	export LD_LIBRARY_PATH=/usr/local/lib
endif

#------ Linux configuration ------

ifeq "$(shell uname -s)" "Linux"

	PLATFORM := Linux
	C_CXX_FLAGS += -DTC_UNIX -DTC_LINUX
	LFLAGS += -rdynamic
	
	# PCSC
	C_CXX_FLAGS += $(shell $(PKG_CONFIG) --cflags libpcsclite)

	# Extract the major and minor version numbers of GCC in a combined format for easy comparison
	GCC_VERSION := $(shell $(CC) -dumpversion | awk -F. '{printf "%d%02d", $$1, $$2}')

	# Set the C++ standard based on the version numbers
	ifeq ($(shell expr $(GCC_VERSION) \< 408), 1)
		# GCC version below 4.8 support minimal C++11 features through the switch -std=c++0x
		CXXFLAGS += -std=c++0x
	else ifeq ($(GCC_VERSION), 408)
		# GCC version 4.8 supports C++11 features through the switch -std=c++11
		CXXFLAGS += -std=c++11
	else ifeq ($(shell expr $(GCC_VERSION) \>= 1100), 1)
		# GNU GCC version 11 and higher compile with -std=gnu++17 by default
		# which breaks "byte" definitions in Crypto++ library. So set -std=gnu++14 instead.
		CXXFLAGS += -std=gnu++14
	endif

	# Linked in GCC versions below 6 was setting large value for MAXPAGESIZE which is not good for ASLR security
	# So, we need to manually add the linker flag "-z max-page-size=4096" to set the maximum page size to 4KB
	# in order to improve ASLR security. Starting from GCC 6, the default value of MAXPAGESIZE is 4KB.
	ifeq ($(shell expr $(GCC_VERSION) \< 600), 1)
		LFLAGS += -Wl,-z,max-page-size=4096
	endif
	
	ifeq "$(SIMD_SUPPORTED)" "1"
		CFLAGS += -msse2
		CXXFLAGS += -msse2

		GCC_GTEQ_440 := $(shell expr `$(CC) -dumpversion | sed -e 's/\.\([0-9][0-9]\)/\1/g' -e 's/\.\([0-9]\)/0\1/g' -e 's/^[0-9]\{3,4\}$$/&00/' -e 's/^[0-9]\{1,2\}$$/&0000/'` \>= 40400)
		GCC_GTEQ_430 := $(shell expr `$(CC) -dumpversion | sed -e 's/\.\([0-9][0-9]\)/\1/g' -e 's/\.\([0-9]\)/0\1/g' -e 's/^[0-9]\{3,4\}$$/&00/' -e 's/^[0-9]\{1,2\}$$/&0000/'` \>= 40300)
		GCC_GTEQ_470 := $(shell expr `$(CC) -dumpversion | sed -e 's/\.\([0-9][0-9]\)/\1/g' -e 's/\.\([0-9]\)/0\1/g' -e 's/^[0-9]\{3,4\}$$/&00/' -e 's/^[0-9]\{1,2\}$$/&0000/'` \>= 40700)
		GCC_GTEQ_500 := $(shell expr `$(CC) -dumpversion | sed -e 's/\.\([0-9][0-9]\)/\1/g' -e 's/\.\([0-9]\)/0\1/g' -e 's/^[0-9]\{3,4\}$$/&00/' -e 's/^[0-9]\{1,2\}$$/&0000/'` \>= 50000)

		ifeq "$(DISABLE_AESNI)" "1"
			CFLAGS += -mno-aes -DCRYPTOPP_DISABLE_AESNI
			CXXFLAGS += -mno-aes -DCRYPTOPP_DISABLE_AESNI
		else
			ifeq "$(GCC_GTEQ_440)" "1"
				CFLAGS += -maes
				CXXFLAGS += -maes
			endif
		endif

		ifeq "$(GCC_GTEQ_430)" "1"
			ifeq "$(origin SSSE3)" "command line"
				CFLAGS += -mssse3
				CXXFLAGS += -mssse3
			endif

			ifeq "$(origin SSE41)" "command line"
				CFLAGS += -mssse3 -msse4.1
				CXXFLAGS += -mssse3 -msse4.1
			endif
		endif
	endif

	ifeq "$(TC_BUILD_CONFIG)" "Release"
		C_CXX_FLAGS += -fdata-sections -ffunction-sections -fpie
		LFLAGS += -Wl,--gc-sections -pie

		ifneq "$(shell ld --help 2>&1 | grep sysv | wc -l)" "0"
			LFLAGS += -Wl,--hash-style=sysv
		endif

		WXCONFIG_CFLAGS += -fdata-sections -ffunction-sections -fpie
		WXCONFIG_CXXFLAGS += -fdata-sections -ffunction-sections -fpie
	endif

	ifneq "$(origin WXSTATIC)" "command line"
		LFLAGS += -ldl
	else
		GCC5USED := $(shell expr `$(CC) -dumpversion | cut -f1 -d.` \>= 5)
		ifeq "$(GCC5USED)" "1"
			CXXFLAGS += -D_GLIBCXX_USE_CXX11_ABI=0
			WXCONFIG_CXXFLAGS += -D_GLIBCXX_USE_CXX11_ABI=0
		endif
	endif
	
	ifeq "$(origin NOSSE2)" "command line"
		CFLAGS += -mno-sse2
		CXXFLAGS += -mno-sse2
		WXCONFIG_CFLAGS += -mno-sse2
		WXCONFIG_CXXFLAGS += -mno-sse2
	endif
endif

#------ Mac OS X configuration ------

ifeq "$(shell uname -s)" "Darwin"

	PLATFORM := MacOSX
	APPNAME := VeraCrypt

	export VC_OSX_TARGET ?= 12
	# use the output of the command "xcrun --show-sdk-version" to set the SDK version if not set
	export VC_OSX_SDK ?= $(shell xcrun --show-sdk-version)

	#check to see if XCode 3 path exists.Otherwise, use XCode 4 path
	VC_OSX_SDK_PATH := /Developer/SDKs/MacOSX$(VC_OSX_SDK).sdk
	ifeq ($(wildcard $(VC_OSX_SDK_PATH)/SDKSettings.plist),)
		VC_OSX_SDK_PATH := $(shell xcrun --sdk macosx$(VC_OSX_SDK) --show-sdk-path)
		ifeq ($(VC_OSX_SDK_PATH),)
$(error Specified SDK version was not found, ensure your active developer directory is correct through xcode-select)
		endif
	endif
	
	#----- Legacy build if OSX <= 10.8: we build both 32-bit and 64-bit ----
	ifneq (,$(filter 10.6 10.7 10.8,$(VC_OSX_TARGET)))
		export VC_LEGACY_BUILD := 1
	endif

	CC := gcc
	CXX := g++

	GCC_GTEQ_440 := 1
	GCC_GTEQ_430 := 1
	GCC_GTEQ_470 := 1
	GCC_GTEQ_500 := 1

	CXXFLAGS += -std=c++11
	C_CXX_FLAGS += -DTC_UNIX -DTC_BSD -DTC_MACOSX -mmacosx-version-min=$(VC_OSX_TARGET) -isysroot $(VC_OSX_SDK_PATH)
	LFLAGS += -mmacosx-version-min=$(VC_OSX_TARGET) -Wl,-syslibroot $(VC_OSX_SDK_PATH) -Wl,-export_dynamic
	# Xcode 15 linker emits a warning "no platform load command found" when linking object files generated by yasm
	# To suppress this warning, we need to use -Wl,-ld_classic flag in order to use the old ld64 linker
	# https://mjtsai.com/blog/2024/03/15/xcode-15-no-platform-load-command-found/
	# We can check whether newer linker is in use if ld -v reports dyld instead of ld64.
	ifeq ($(shell xcrun --sdk macosx$(VC_OSX_SDK) ld -v 2>&1 | grep -oE 'PROJECT:[^-]+' | cut -d: -f2),dyld)
		LFLAGS += -Wl,-ld_classic
	endif

	WX_CONFIGURE_FLAGS += --with-macosx-version-min=$(VC_OSX_TARGET) --with-macosx-sdk=$(VC_OSX_SDK_PATH)

	ifneq "$(VC_OSX_FUSET)" "0"
		C_CXX_FLAGS += -DVC_MACOSX_FUSET
		VC_FUSE_PACKAGE := fuse-t
	endif

	export CFLAGS_ARM64 := $(CFLAGS) $(C_CXX_FLAGS) -arch arm64 -march=armv8-a+crypto
	export CFLAGS_X64 := $(CFLAGS) $(C_CXX_FLAGS) -arch x86_64

	# Set x86 assembly flags (-msse2, -mssse3, -msse4.1)
	# Apply flags if SIMD_SUPPORTED is 1 or if not in local development build (we are creating universal binary in this case)
	ifneq "$(LOCAL_DEVELOPMENT_BUILD)" "true"
		SIMD_SUPPORTED = 1
	endif

	ifeq "$(SIMD_SUPPORTED)" "1"
		CFLAGS += -msse2
		CXXFLAGS += -msse2

		ifeq "$(origin SSSE3)" "command line"
			CFLAGS += -mssse3
			CXXFLAGS += -mssse3
		endif

		ifeq "$(origin SSE41)" "command line"
			CFLAGS += -mssse3 -msse4.1
			CXXFLAGS += -mssse3 -msse4.1
		endif
	endif

	AS ?= $(BASE_DIR)/Build/Tools/MacOSX/yasm
	export ASFLAGS32 := -D __GNUC__ -D __YASM__ -D __BITS__=32 --prefix=_ -f macho32
	export ASFLAGS64 := -D __GNUC__ -D __YASM__ -D __BITS__=64 --prefix=_ -f macho64

	ifeq "$(TC_BUILD_CONFIG)" "Release"

    	export DISABLE_PRECOMPILED_HEADERS := 1

		C_CXX_FLAGS := $(subst -MMD,,$(C_CXX_FLAGS)) -gfull
		LFLAGS += -Wl,-dead_strip

		# Initialize architecture flag
		ARCH_FLAG := -arch x86_64

		# Set architecture flags based on build type and CPU architecture
		ifeq "$(LOCAL_DEVELOPMENT_BUILD)" "true"
			ifeq "$(CPU_ARCH)" "arm64"
				ARCH_FLAG := -arch arm64
			endif
			WX_CONFIGURE_FLAGS += --disable-universal_binary
		else
			# Legacy build settings
			ifdef VC_LEGACY_BUILD
				ARCH_FLAG += -arch i386
				WX_CONFIGURE_FLAGS += --enable-universal_binary=i386,x86_64
			else
				# Non-development build defaults to universal binary for arm64 and x86_64
				ARCH_FLAG += -arch arm64
				WX_CONFIGURE_FLAGS += --enable-universal_binary=arm64,x86_64
			endif
		endif

		# Apply architecture flags
		C_CXX_FLAGS += $(ARCH_FLAG)
		LFLAGS += $(ARCH_FLAG)

		WX_CONFIGURE_FLAGS += --without-libpng --disable-gif --disable-pcx --disable-tga --disable-iff --disable-svg
		WXCONFIG_CFLAGS += -gfull
		WXCONFIG_CXXFLAGS += -gfull

	else

		WX_CONFIGURE_FLAGS += --disable-universal_binary

	endif

endif


#------ FreeBSD configuration ------

ifeq "$(shell uname -s)" "FreeBSD"

	PLATFORM := FreeBSD
	PLATFORM_UNSUPPORTED := 1
	C_CXX_FLAGS += -DTC_UNIX -DTC_BSD -DTC_FREEBSD
	LFLAGS += -rdynamic

	# PCSC
	C_CXX_FLAGS += $(shell $(PKG_CONFIG) --cflags libpcsclite)
	
	CC := cc
	CXX := c++

	GCC_GTEQ_440 := 1
	GCC_GTEQ_430 := 1
	GCC_GTEQ_470 := 1
	GCC_GTEQ_500 := 1
	
	ifeq "$(TC_BUILD_CONFIG)" "Release"
		C_CXX_FLAGS += -fdata-sections -ffunction-sections -fpie
		LFLAGS += -Wl,--gc-sections -pie

		ifneq "$(shell ld --help 2>&1 | grep sysv | wc -l)" "0"
			LFLAGS += -Wl,--hash-style=sysv
		endif

		WXCONFIG_CFLAGS += -fpie -fPIC
		WXCONFIG_CXXFLAGS += -fpie -fPIC
	endif

	ifeq "$(SIMD_SUPPORTED)" "1"
		CFLAGS += -msse2
		CXXFLAGS += -msse2
		
		ifeq "$(DISABLE_AESNI)" "1"
			CFLAGS += -mno-aes -DCRYPTOPP_DISABLE_AESNI
			CXXFLAGS += -mno-aes -DCRYPTOPP_DISABLE_AESNI
		else
			CFLAGS += -maes
			CXXFLAGS += -maes
		endif

		ifeq "$(origin SSSE3)" "command line"
			CFLAGS += -mssse3
			CXXFLAGS += -mssse3
		endif

		ifeq "$(origin SSE41)" "command line"
			CFLAGS += -mssse3 -msse4.1
			CXXFLAGS += -mssse3 -msse4.1
		endif
	endif
	
	ifeq "$(origin NOSSE2)" "command line"
		CFLAGS += -mno-sse2
		CXXFLAGS += -mno-sse2
		WXCONFIG_CFLAGS += -mno-sse2
		WXCONFIG_CXXFLAGS += -mno-sse2
	endif

endif


#------ OpenBSD configuration ------

ifeq "$(shell uname -s)" "OpenBSD"

	PLATFORM := OpenBSD
	PLATFORM_UNSUPPORTED := 1
	C_CXX_FLAGS += -DTC_UNIX -DTC_BSD -DTC_OPENBSD
	LFLAGS += -rdynamic

	# PCSC
	C_CXX_FLAGS += $(shell $(PKG_CONFIG) --cflags libpcsclite)

	CC := cc
	CXX := c++

	GCC_GTEQ_440 := 1
	GCC_GTEQ_430 := 1
	GCC_GTEQ_470 := 1
	GCC_GTEQ_500 := 1

	ifeq "$(TC_BUILD_CONFIG)" "Release"
		C_CXX_FLAGS += -fdata-sections -ffunction-sections -fpie
		LFLAGS += -Wl,--gc-sections -pie

		WXCONFIG_CFLAGS += -fpie -fPIC
		WXCONFIG_CXXFLAGS += -fpie -fPIC
	endif
endif


#------ Solaris configuration ------

ifeq "$(shell uname -s)" "SunOS"

	PLATFORM := Solaris
	PLATFORM_UNSUPPORTED := 1
	C_CXX_FLAGS += -DTC_UNIX -DTC_SOLARIS
	WX_CONFIGURE_FLAGS += --with-gtk

	# PCSC
	C_CXX_FLAGS += $(shell $(PKG_CONFIG) --cflags libpcsclite)

endif

ifneq (,$(filter Linux FreeBSD OpenBSD,$(PLATFORM)))
	# Determine GTK version
	GTK_VERSION := $(shell $(PKG_CONFIG) --modversion gtk+-3.0 2>/dev/null | grep -o '^3' || echo 2)
	ifeq ($(GTK_VERSION),3)
		WX_CONFIGURE_FLAGS += --with-gtk=3
	else
		WX_CONFIGURE_FLAGS += --with-gtk=2
	endif

	ifeq "$(origin INDICATOR)" "command line"
		ifeq ($(GTK_VERSION),3)
			INDICATOR_LIBRARY=ayatana-appindicator3-0.1
		else
			INDICATOR_LIBRARY=ayatana-appindicator-0.1
		endif
		export AYATANA_LIBS += $(shell $(PKG_CONFIG) --libs $(INDICATOR_LIBRARY))
		C_CXX_FLAGS += $(shell $(PKG_CONFIG) --cflags $(INDICATOR_LIBRARY)) -DHAVE_INDICATORS
	endif
endif

#------ Common configuration ------

CFLAGS := $(C_CXX_FLAGS) $(CFLAGS) $(TC_EXTRA_CFLAGS)
CXXFLAGS := $(C_CXX_FLAGS) $(CXXFLAGS) $(TC_EXTRA_CXXFLAGS)
LFLAGS := $(LFLAGS) $(TC_EXTRA_LFLAGS)

WX_CONFIGURE_FLAGS += --disable-shared --disable-dependency-tracking --enable-exceptions --enable-dataobj --enable-mimetype

ifdef VC_WX_MINIMAL
WX_CONFIGURE_FLAGS += --disable-protocol --disable-protocols --disable-url --disable-ipc --disable-sockets --without-libcurl --disable-fs_inet --disable-ole --disable-docview --disable-clipboard \
	--disable-help --disable-html --disable-mshtmlhelp --disable-htmlhelp --disable-mdi --disable-metafile --disable-addremovectrl --disable-webview \
	--disable-xrc --disable-aui --disable-postscript --disable-printarch \
	--disable-arcstream --disable-fs_archive --disable-fs_zip --disable-tarstream --disable-zipstream \
	--disable-animatectrl --disable-bmpcombobox --disable-calendar --disable-caret --disable-checklst --disable-collpane --disable-colourpicker --disable-comboctrl \
	--disable-datepick --disable-display --disable-dirpicker --disable-filepicker --disable-fontpicker --disable-grid --disable-dataviewctrl \
	--disable-listbook --disable-odcombobox --disable-sash --disable-searchctrl --disable-slider --disable-splitter --disable-togglebtn \
	--disable-toolbar --disable-tbarnative --disable-treebook --disable-toolbook --disable-tipwindow --disable-popupwin \
	--disable-commondlg --disable-aboutdlg --disable-coldlg --disable-finddlg --disable-fontdlg --disable-numberdlg --disable-splash \
	--disable-tipdlg --disable-progressdlg --disable-wizarddlg --disable-miniframe --disable-splines --disable-palette \
	--disable-richtext --disable-dialupman --disable-debugreport --disable-filesystem --disable-rearrangectrl --disable-treelist --disable-richmsgdlg \
	--disable-richtooltip --disable-propgrid --disable-stc --without-libnotify \
	--without-gtkprint --without-gnomevfs --disable-fsvolume --disable-fswatcher \
	--disable-sound --disable-mediactrl --disable-joystick --disable-apple_ieee \
	--disable-gif --disable-pcx --disable-tga --disable-iff --disable-gif --disable-pnm --disable-svg \
	--without-expat --without-libtiff --without-libjpeg --without-libpng -without-regex --without-zlib
	
ifneq (,$(filter Linux FreeBSD,$(PLATFORM)))
WX_CONFIGURE_FLAGS += --disable-tooltips
ifneq ($(GTK_VERSION),3)
	WX_CONFIGURE_FLAGS += --disable-graphics_ctx
endif
else
	WX_CONFIGURE_FLAGS += --disable-graphics_ctx
endif
else
# Disable libtiff on macOS
ifeq "$(PLATFORM)" "MacOSX"
	WX_CONFIGURE_FLAGS += --without-libtiff
endif
endif


#------ Project build ------

PROJ_DIRS := Platform Volume Driver/Fuse Core Main

.PHONY: all clean wxbuild

all clean:
	@if pwd | grep -q ' '; then echo 'Error: source code is stored in a path containing spaces' >&2; exit 1; fi

	@for DIR in $(PROJ_DIRS); do \
		PROJ=$$(echo $$DIR | cut -d/ -f1); \
		$(MAKE) -C $$DIR -f $$PROJ.make NAME=$$PROJ $(MAKECMDGOALS) || exit $?; \
		export LIBS="$(BASE_DIR)/$$DIR/$$PROJ.a $$LIBS"; \
	done

install:
	$(MAKE) -C Main -f Main.make NAME=Main install

package:
	$(MAKE) -C Main -f Main.make NAME=Main package

appimage:
	$(MAKE) -C Main -f Main.make NAME=Main appimage

#------ wxWidgets build ------

ifeq "$(MAKECMDGOALS)" "wxbuild"
CFLAGS :=
CXXFLAGS :=
LFLAGS :=
endif

wxbuild:

ifneq "$(shell test -f $(WX_ROOT)/configure || test -f $(WX_BUILD_DIR)/../configure && echo 1)" "1"
	@echo 'Error: WX_ROOT must point to wxWidgets source code directory' >&2
	@exit 1
endif

	rm -rf "$(WX_BUILD_DIR)"
	mkdir -p "$(WX_BUILD_DIR)"
	@echo Configuring wxWidgets library...
	cd "$(WX_BUILD_DIR)" && "$(WX_ROOT)/configure" $(WX_CONFIGURE_FLAGS) >/dev/null

	@echo Building wxWidgets library...
	cd "$(WX_BUILD_DIR)" && $(MAKE) -j 4
