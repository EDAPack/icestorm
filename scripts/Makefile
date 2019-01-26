
SCRIPTS_DIR:=$(abspath $(dir $(lastword $(MAKEFILE_LIST))))
PROJECT_DIR:=$(abspath $(SCRIPTS_DIR)/..)
PACKAGES_DIR:=$(PROJECT_DIR)/packages
BUILD_DIR:=$(PROJECT_DIR)/build
ETC_DIR:=$(PROJECT_DIR)/etc

EDAPACK_BUILD_URL=https://github.com/EDAPack/edapack-build

include $(PROJECT_DIR)/etc/package.info
name:=$(name)
version:=$(version)

ifeq (true,$(VERBOSE))
Q=
WGET=wget
UNTARGZ=tar xvzf
UNTARBZ=tar xvjf
UNZIP=unzip -o
else
Q=@
WGET=wget -q
UNTARGZ=tar xzf
UNTARBZ=tar xjf
UNZIP=unzip -o -q
endif

uname_o:=$(shell uname -o)

ifeq (Linux,$(uname_o))
platform=linux_x86_64
else
ifeq (GNU/Linux,$(uname_o))
platform=linux_x86_64
else
platform=unknown
endif
endif

# TODO: Package TGZ and URL
ICESTORM_DIR=icestorm-master
ICESTORM_ZIP=$(ICESTORM_DIR).zip
ICESTORM_URL=https://github.com/cliffordwolf/icestorm/archive/master.zip

ARACHNE_PNR_DIR=arachne-pnr-master
ARACHNE_PNR_ZIP=$(ARACHNE_PNR_DIR).zip
ARACHNE_PNR_URL=https://github.com/cseed/arachne-pnr/archive/master.zip

NEXTPNR_DIR=nextpnr-master
NEXTPNR_ZIP=$(NEXTPNR_DIR).zip
NEXTPNR_URL=https://github.com/YosysHQ/nextpnr/archive/master.zip

YOSYS_DIR=yosys-master
YOSYS_ZIP=$(YOSYS_DIR).zip
YOSYS_URL=https://github.com/YosysHQ/yosys/archive/master.zip

READLINE_VERSION=7.0
READLINE_DIR=readline-$(READLINE_VERSION)
READLINE_TGZ=$(READLINE_DIR).tar.gz
READLINE_URL=https://ftp.gnu.org/gnu/readline/$(READLINE_TGZ)

FFI_VERSION=3.2.1
FFI_DIR=libffi-$(FFI_VERSION)
FFI_TGZ=$(FFI_DIR).tar.gz
FFI_URL=ftp://sourceware.org/pub/libffi/$(FFI_TGZ)

ABC_DIR=abc-master
ABC_ZIP=$(ABC_DIR).zip
ABC_URL=https://github.com/berkeley-abc/abc/archive/master.zip

LIBFTDI_VERSION=-1.4
LIBFTDI_DIR=libftdi1$(LIBFTDI_VERSION)
LIBFTDI_TBZ=$(LIBFTDI_DIR).tar.bz2
LIBFTDI_URL=https://www.intra2net.com/en/developer/libftdi/download/libftdi1-1.4.tar.bz2

CONFUSE_VERSION=2.5
CONFUSE_DIR=confuse-$(CONFUSE_VERSION)
CONFUSE_TGZ=$(CONFUSE_DIR).tar.gz
CONFUSE_URL=https://www.intra2net.com/en/developer/libftdi/download/$(CONFUSE_TGZ)

LIBUSB_VERSION=1.0.22
LIBUSB_DIR=libusb-$(LIBUSB_VERSION)
LIBUSB_TBZ=$(LIBUSB_DIR).tar.bz2
LIBUSB_URL=https://github.com/libusb/libusb/releases/download/v$(LIBUSB_VERSION)/$(LIBUSB_TBZ)

PREFIX=$(BUILD_DIR)/$(name)_inst
INST_PREFIX=$(PREFIX)/$(name)
PACKAGE=$(BUILD_DIR)/$(name)-$(platform)-$(version).tar.gz

all : $(PACKAGE)

$(PACKAGE) : \
		$(BUILD_DIR)/icestorm.d \
		$(BUILD_DIR)/arachne-pnr.d \
		$(BUILD_DIR)/nextpnr.d \
		$(BUILD_DIR)/yosys.d \
		$(BUILD_DIR)/stdcpp.d \
		$(PACKAGES_DIR)/install.py \
		$(ETC_DIR)/modulefile \
		$(ETC_DIR)/package.info
	$(Q)mkdir -p $(PREFIX)/etc
	$(Q)cp $(PACKAGES_DIR)/install.py $(PREFIX)/etc
	$(Q)sed -e "s/{{version}}/$(version)/g" \
		-e "s/{{name}}/$(name)/g" \
		$(ETC_DIR)/modulefile  > $(PREFIX)/etc/modulefile
	$(Q)cp $(ETC_DIR)/package.info $(PREFIX)/etc
	$(Q)mkdir -p $(PREFIX)/bin
	$(Q)for exe in $(INST_PREFIX)/bin/*; do \
		cp $(SCRIPTS_DIR)/launcher $(PREFIX)/bin/`basename $$exe` ; \
		chmod +x $(PREFIX)/bin/`basename $$exe`; \
	done
	$(Q)cd $(PREFIX) ; tar czf $@ *

WHICH_LIBSTDCPP:=$(shell $(CXX) -print-file-name=libstdc++.so)

$(BUILD_DIR)/stdcpp.d :
	$(Q)mkdir -p $(INST_PREFIX)/lib
	$(Q)cd `dirname $(WHICH_LIBSTDCPP)`; \
		tar cf - libstdc++.so* | (cd $(INST_PREFIX)/lib ; tar xf -)
	$(Q)touch $@

$(BUILD_DIR)/icestorm.d : \
		$(PACKAGES_DIR)/$(ICESTORM_ZIP) \
		$(BUILD_DIR)/libftdi.d
	$(Q)rm -rf $(BUILD_DIR)/icestorm
	$(Q)mkdir -p $(BUILD_DIR)/icestorm
	$(Q)cd $(BUILD_DIR)/icestorm ; $(UNZIP) $(PACKAGES_DIR)/$(ICESTORM_ZIP)
	$(Q)cd $(BUILD_DIR)/icestorm/$(ICESTORM_DIR) ; \
		$(MAKE) PREFIX=$(INST_PREFIX) \
			LDFLAGS="-L$(INST_PREFIX)/lib -lusb-1.0 -lrt" \
			PKG_CONFIG_PATH=$(INST_PREFIX)/lib/pkgconfig
	$(Q)cd $(BUILD_DIR)/icestorm/$(ICESTORM_DIR) ; \
		$(MAKE) PREFIX=$(INST_PREFIX) install
	$(Q)touch $@

$(BUILD_DIR)/arachne-pnr.d : \
		$(PACKAGES_DIR)/$(ARACHNE_PNR_ZIP) \
		$(BUILD_DIR)/icestorm.d
	$(Q)rm -rf $(BUILD_DIR)/arachne-pnr
	$(Q)mkdir -p $(BUILD_DIR)/arachne-pnr
	$(Q)cd $(BUILD_DIR)/arachne-pnr ; \
		$(UNZIP) $(PACKAGES_DIR)/$(ARACHNE_PNR_ZIP)
	$(Q)cd $(BUILD_DIR)/arachne-pnr/$(ARACHNE_PNR_DIR) ; \
		$(MAKE) PREFIX=$(INST_PREFIX)
	$(Q)cd $(BUILD_DIR)/arachne-pnr/$(ARACHNE_PNR_DIR) ; \
		$(MAKE) PREFIX=$(INST_PREFIX) install
	$(Q)touch $@

$(BUILD_DIR)/nextpnr.d : \
		$(PACKAGES_DIR)/$(NEXTPNR_ZIP) \
		$(BUILD_DIR)/icestorm.d
	$(Q)rm -rf $(BUILD_DIR)/nextpnr
	$(Q)mkdir -p $(BUILD_DIR)/nextpnr
	$(Q)cd $(BUILD_DIR)/nextpnr ; \
		$(UNZIP) $(PACKAGES_DIR)/$(NEXTPNR_ZIP)
	$(Q)cd $(BUILD_DIR)/nextpnr/$(NEXTPNR_DIR) ; \
		cmake -DARCH=ice40 -DCMAKE_INSTALL_PREFIX=$(INST_PREFIX) \
			-DICEBOX_ROOT=$(INST_PREFIX)/share/icebox \
			-DBUILD_GUI=OFF \
			-DCMAKE_CXX_COMPILER=$(shell which $(CXX))
	$(Q)cd $(BUILD_DIR)/nextpnr/$(NEXTPNR_DIR) ; $(MAKE)
	$(Q)cd $(BUILD_DIR)/nextpnr/$(NEXTPNR_DIR) ; $(MAKE) install
	$(Q)touch $@

# TODO: Fetch package TGZ
$(PACKAGES_DIR)/$(ICESTORM_ZIP) :
	$(Q)$(WGET) -O $@ $(ICESTORM_URL) || (rm -f $@ ; exit 1)

$(PACKAGES_DIR)/$(ARACHNE_PNR_ZIP) :
	$(Q)$(WGET) -O $@ $(ARACHNE_PNR_URL) || (rm -f $@ ; exit 1)

$(PACKAGES_DIR)/$(NEXTPNR_ZIP) :
	$(Q)$(WGET) -O $@ $(NEXTPNR_URL) || (rm -f $@ ; exit 1)

$(BUILD_DIR)/libftdi.d : \
		$(PACKAGES_DIR)/$(LIBFTDI_TBZ) \
		$(BUILD_DIR)/confuse.d \
		$(BUILD_DIR)/libusb.d
	$(Q)rm -rf $(BUILD_DIR)/libftdi
	$(Q)mkdir -p $(BUILD_DIR)/libftdi
	$(Q)cd $(BUILD_DIR)/libftdi ; $(UNTARBZ) $(PACKAGES_DIR)/$(LIBFTDI_TBZ)
	$(Q)mkdir -p $(BUILD_DIR)/libftdi/$(LIBFTDI_DIR)/build
	$(Q)cd $(BUILD_DIR)/libftdi/$(LIBFTDI_DIR)/build ; \
		cmake -DCMAKE_INSTALL_PREFIX=$(INST_PREFIX) \
			-DCMAKE_CXX_COMPILER=$(shell which $(CXX)) \
			-DCMAKE_EXE_LINKER_FLAGS=-lrt \
			-DLIBUSB_INCLUDE_DIR=$(INST_PREFIX)/include/libusb-1.0 \
			-DLIBUSB_LIBRARIES=$(INST_PREFIX)/lib/libusb-1.0.so \
			-DSTATICLIBS=OFF ..
	$(Q)cd $(BUILD_DIR)/libftdi/$(LIBFTDI_DIR)/build ; $(MAKE)
	$(Q)cd $(BUILD_DIR)/libftdi/$(LIBFTDI_DIR)/build ; $(MAKE) install
	$(Q)cp $(INST_PREFIX)/include/libftdi1/* $(INST_PREFIX)/include
	$(Q)mv $(INST_PREFIX)/lib64/*.so* $(INST_PREFIX)/lib
	$(Q)cd $(INST_PREFIX)/lib ; ln -s libftdi1.so libftdi.so
	$(Q)touch $@

$(PACKAGES_DIR)/$(LIBFTDI_TBZ) : 
	$(Q)mkdir -p $(PACKAGES_DIR)
	$(Q)$(WGET) -O $@ $(LIBFTDI_URL) || (rm -f $@ ; exit 1)

$(BUILD_DIR)/confuse.d : $(PACKAGES_DIR)/$(CONFUSE_TGZ)
	$(Q)rm -rf $(BUILD_DIR)/confuse
	$(Q)mkdir -p $(BUILD_DIR)/confuse
	$(Q)cd $(BUILD_DIR)/confuse ; $(UNTARGZ) $(PACKAGES_DIR)/$(CONFUSE_TGZ)
	$(Q)cd $(BUILD_DIR)/confuse/$(CONFUSE_DIR) ; \
		./configure --prefix=$(INST_PREFIX)
	$(Q)cd $(BUILD_DIR)/confuse/$(CONFUSE_DIR) ; $(MAKE)
	$(Q)cd $(BUILD_DIR)/confuse/$(CONFUSE_DIR) ; $(MAKE) install
	$(Q)touch $@

$(PACKAGES_DIR)/$(CONFUSE_TGZ) : 
	$(Q)mkdir -p $(PACKAGES_DIR)
	$(Q)$(WGET) -O $@ $(CONFUSE_URL) || (rm -f $@ ; exit 1)

$(BUILD_DIR)/libusb.d : $(PACKAGES_DIR)/$(LIBUSB_TBZ)
	$(Q)rm -rf $(BUILD_DIR)/libusb
	$(Q)mkdir -p $(BUILD_DIR)/libusb
	$(Q)cd $(BUILD_DIR)/libusb ; $(UNTARBZ) $(PACKAGES_DIR)/$(LIBUSB_TBZ)
	$(Q)cd $(BUILD_DIR)/libusb/$(LIBUSB_DIR) ; \
		./configure --prefix=$(INST_PREFIX) --enable-shared
	$(Q)cd $(BUILD_DIR)/libusb/$(LIBUSB_DIR) ; $(MAKE)
	$(Q)cd $(BUILD_DIR)/libusb/$(LIBUSB_DIR) ; $(MAKE) install
	$(Q)touch $@

$(PACKAGES_DIR)/$(LIBUSB_TBZ) : 
	$(Q)mkdir -p $(PACKAGES_DIR)
	$(Q)$(WGET) -O $@ $(LIBUSB_URL) || (rm -f $@ ; exit 1)

YOSYS_EXTRA_CXXFLAGS += -I$(INST_PREFIX)/lib/$(FFI_DIR)/include
YOSYS_EXTRA_LDFLAGS += -L$(INST_PREFIX)/lib
YOSYS_EXTRA_LDFLAGS += -L$(INST_PREFIX)/lib64

$(BUILD_DIR)/yosys.d : \
		$(PACKAGES_DIR)/$(YOSYS_ZIP) \
		$(PACKAGES_DIR)/$(ABC_ZIP) \
		$(PACKAGES_DIR)/$(READLINE_TGZ) \
		$(PACKAGES_DIR)/$(FFI_TGZ)
	$(Q)echo SCRIPTS_DIR=$(SCRIPTS_DIR)
	$(Q)rm -rf $(BUILD_DIR)/yosys
	$(Q)mkdir -p $(BUILD_DIR)/yosys
	$(Q)cd $(BUILD_DIR)/yosys ; $(UNTARGZ) $(PACKAGES_DIR)/$(FFI_TGZ)
	$(Q)cd $(BUILD_DIR)/yosys/$(FFI_DIR) ; \
		./configure --disable-shared \
			--prefix=$(INST_PREFIX)
	$(Q)cd $(BUILD_DIR)/yosys/$(FFI_DIR) ; $(MAKE)
	$(Q)cd $(BUILD_DIR)/yosys/$(FFI_DIR) ; $(MAKE) install
	$(Q)cd $(BUILD_DIR)/yosys ; $(UNZIP) $(PACKAGES_DIR)/$(YOSYS_ZIP)
	$(Q)cd $(BUILD_DIR)/yosys ; $(UNZIP) $(PACKAGES_DIR)/$(ABC_ZIP)
	$(Q)cd $(BUILD_DIR)/yosys ; mv $(ABC_DIR) $(YOSYS_DIR)/abc
	$(Q)cd $(BUILD_DIR)/yosys/$(YOSYS_DIR) ; \
		patch -p1 < $(SCRIPTS_DIR)/yosys.patch
	$(Q)cd $(BUILD_DIR)/yosys/$(YOSYS_DIR) ; \
		sed -i \
			-e 's/CONFIG := clang/CONFIG := gcc/g' \
			-e 's/ENABLE_READLINE := 1/ENABLE_READLINE := 0/g' \
			-e 's/ENABLE_TCL := 1/ENABLE_TCL := 0/g' \
			-e 's/CXXFLAGS := \$$(CXXFLAGS)/CXXFLAGS := \$$(CXXFLAGS) \$$(EXTRA_CXXFLAGS)/g' Makefile \
			-e 's/LDFLAGS := \$$(LDFLAGS)/LDFLAGS := \$$(LDFLAGS) \$$(EXTRA_LDFLAGS)/g' Makefile
	$(Q)if test "x$(VERBOSE)" = "xtrue"; then \
		cd $(BUILD_DIR)/yosys/$(YOSYS_DIR) ; \
		sed -i -e 's/PRETTY = 1/PRETTY = 0/g' Makefile; \
	fi
	$(Q)cd $(BUILD_DIR)/yosys/$(YOSYS_DIR)/abc ; \
		sed -i -e 's/-Wall/\$$(EXTRA_CXXFLAGS) -Wall/g' \
			-e 's/-lreadline/\$$(EXTRA_LDFLAGS) -lreadline/g' \
		Makefile
	$(Q)cd $(BUILD_DIR)/yosys/$(YOSYS_DIR) ; \
		$(MAKE) PREFIX=$(INST_PREFIX) \
			EXTRA_CXXFLAGS="$(YOSYS_EXTRA_CXXFLAGS)" \
			EXTRA_LDFLAGS="$(YOSYS_EXTRA_LDFLAGS)" \
			ABCPULL=0 ABCREV=default VERBOSE=$(Q)
	$(Q)cd $(BUILD_DIR)/yosys/$(YOSYS_DIR) ; \
		$(MAKE) PREFIX=$(INST_PREFIX) \
			ABCPULL=0 ABCREV=default VERBOSE=$(Q) install
	$(Q)touch $@

$(BUILD_DIR)/readline.d : $(PACKAGES_DIR)/$(READLINE_TGZ)
	$(Q)rm -rf $(BUILD_DIR)/readline
	$(Q)mkdir -p $(BUILD_DIR)/readline
	$(Q)cd $(BUILD_DIR)/readline ; $(UNTARGZ) $(PACKAGES_DIR)/$(READLINE_TGZ)
	$(Q)cd $(BUILD_DIR)/readline/$(READLINE_DIR) ; \
		./configure --disable-shared --prefix=$(BUILD_DIR)/readline/inst
	$(Q)cd $(BUILD_DIR)/readline/$(READLINE_DIR) ; $(MAKE) 
	$(Q)cd $(BUILD_DIR)/readline/$(READLINE_DIR) ; $(MAKE) install
	$(Q)touch $@

$(PACKAGES_DIR)/$(READLINE_TGZ) :
	$(Q)if test ! -d $(PACKAGES_DIR); then mkdir -p $(PACKAGES_DIR); fi
	$(Q)$(WGET) -O $@ $(READLINE_URL)

$(PACKAGES_DIR)/$(FFI_TGZ) :
	$(Q)if test ! -d $(PACKAGES_DIR); then mkdir -p $(PACKAGES_DIR); fi
	$(Q)$(WGET) -O $@ $(FFI_URL)

$(PACKAGES_DIR)/$(YOSYS_ZIP) : 
	$(Q)mkdir -p $(PACKAGES_DIR)
	$(Q)$(WGET) -O $@ $(YOSYS_URL)

$(PACKAGES_DIR)/$(ABC_ZIP) : 
	$(Q)if test ! -d $(PACKAGES_DIR); then mkdir -p $(PACKAGES_DIR); fi
	$(Q)$(WGET) -O $@ $(ABC_URL)

$(PACKAGES_DIR)/install.py : 
	$(Q)mkdir -p $(PACKAGES_DIR)
	$(Q)$(WGET) -O $@ $(EDAPACK_BUILD_URL)/raw/master/scripts/install.py

release : $(PACKAGE) $(PACKAGES_DIR)/upload.py
	$(Q)python3 $(PACKAGES_DIR)/upload.py \
		--org EDAPack --repo $(name) \
		--key $(GITHUB_API_TOKEN) --version $(version) $(PACKAGE)

$(PACKAGES_DIR)/upload.py :
	$(Q)mkdir -p $(PACKAGES_DIR)
	$(Q)$(WGET) -O $@ $(EDAPACK_BUILD_URL)/raw/master/scripts/upload.py

clean : 
	$(Q)rm -rf $(BUILD_DIR)

clean-all : 
	$(Q)rm -rf $(BUILD_DIR) $(PACKAGES_DIR)
