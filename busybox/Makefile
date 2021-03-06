# Build Busybox as follows:
#
# - make               -- create non-SGX no-debug-log manifest
# - make DEBUG=1       -- create non-SGX debug-log manifest
# - make SGX=1         -- create SGX no-debug-log manifest
# - make SGX=1 DEBUG=1 -- create SGX debug-log manifest
#
# Any of these invocations downloads Busybox and builds it.
#
# Use `make clean` to remove Graphene-generated files and `make distclean` to
# additionally remove the Busybox source.

################################# CONSTANTS ###################################

# Relative path to Graphene root
GRAPHENEDIR = ../../../../..

SRCDIR = src
BUSYBOX_URL ?= https://busybox.net/downloads/busybox-1.31.1.tar.bz2
BUSYBOX_SHA256 ?= d0f940a72f648943c1f2211e0e3117387c31d765137d92bd8284a3fb9752a998

ifeq ($(DEBUG),1)
GRAPHENEDEBUG = inline
else
GRAPHENEDEBUG = none
endif


.PHONY: all
all: busybox busybox.manifest pal_loader
ifeq ($(SGX),1)
all: busybox.manifest.sgx
endif

############################ BUSYBOX EXECUTABLE ###############################

# Busybox is built as usual, without any changes to the build process. The
# source is downloaded and then built via classic make. The result of this
# build process is the final executable "src/busybox".

$(SRCDIR)/Makefile:
	$(GRAPHENEDIR)/Scripts/download --output busybox.tar.bz2 \
		--sha256 $(BUSYBOX_SHA256) --url $(BUSYBOX_URL)
	mkdir $(SRCDIR)
	tar -C $(SRCDIR) --strip-components=1 -xf busybox.tar.bz2

$(SRCDIR)/.config: $(SRCDIR)/Makefile
	cd $(SRCDIR) && make clean && make defconfig
	# Enable usage of Busybox's built-in applets
	sed -e 's/.*CONFIG_FEATURE_SH_STANDALONE.*/CONFIG_FEATURE_SH_STANDALONE=y/' \
		-i $(SRCDIR)/.config
	# Currently '/proc/self/exe' is bugged in graphene, so manually set path
	# to the Busybox binary
	sed -e 's/.*CONFIG_BUSYBOX_EXEC_PATH.*/CONFIG_BUSYBOX_EXEC_PATH="\/busybox"/' \
		-i $(SRCDIR)/.config

$(SRCDIR)/busybox: $(SRCDIR)/.config
	cd $(SRCDIR) && make

############################## BUSYBOX MANIFEST ###############################

# The template file contains almost all necessary information to run Busybox
# under Graphene / Graphene-SGX. We create busybox.manifest (to be run under
# non-SGX Graphene) by simply replacing variables in the template file via sed.

busybox.manifest: busybox.manifest.template
	sed -e 's|$$(GRAPHENEDIR)|'"$(GRAPHENEDIR)"'|g' \
		-e 's|$$(GRAPHENEDEBUG)|'"$(GRAPHENEDEBUG)"'|g' \
		$< > $@

# Manifest for Graphene-SGX requires special "pal-sgx-sign" procedure. This
# procedure measures all Busybox dependencies (shared libraries and trusted
# files), measures Busybox code/data pages, and adds measurements into the
# resulting manifest.sgx file (among other, less important SGX options).
#
# Additionally, Graphene-SGX requires EINITTOKEN and SIGSTRUCT objects (see
# SGX hardware ABI, in particular EINIT instruction). The "pal-sgx-get-token"
# script generates these objects and puts them in files .token and .sig
# respectively. Note that filenames must be the same as the executable/manifest
# name (i.e., "busybox").

busybox.manifest.sgx: busybox.manifest busybox
	$(GRAPHENEDIR)/Pal/src/host/Linux-SGX/signer/pal-sgx-sign \
		-libpal $(GRAPHENEDIR)/Runtime/libpal-Linux-SGX.so \
		-key $(GRAPHENEDIR)/Pal/src/host/Linux-SGX/signer/enclave-key.pem \
		-manifest $< -output $@ \
		-exec busybox
	$(GRAPHENEDIR)/Pal/src/host/Linux-SGX/signer/pal-sgx-get-token \
		-output busybox.token -sig busybox.sig

########################### COPIES OF EXECUTABLES #############################

# Busybox build process creates the final executable as "src/busybox".
# Copy Busybox binary to our root directory and create a link to pal_loader for
# simplicity.

busybox: $(SRCDIR)/busybox
	cp $(SRCDIR)/busybox busybox

pal_loader:
	ln -s $(GRAPHENEDIR)/Runtime/pal_loader $@

################################## CLEANUP ####################################

.PHONY: clean
clean:
	$(RM) *.token *.sig *.manifest.sgx *.manifest pal_loader busybox

.PHONY: distclean
distclean: clean
	$(RM) -r $(SRCDIR) busybox.tar.bz2

