# Makefile
#
# @Copyright@
# 
# 				Rocks(r)
# 		         www.rocksclusters.org
# 		         version 6.2 (SideWinder)
# 
# Copyright (c) 2000 - 2014 The Regents of the University of California.
# All rights reserved.	
# 
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions are
# met:
# 
# 1. Redistributions of source code must retain the above copyright
# notice, this list of conditions and the following disclaimer.
# 
# 2. Redistributions in binary form must reproduce the above copyright
# notice unmodified and in its entirety, this list of conditions and the
# following disclaimer in the documentation and/or other materials provided 
# with the distribution.
# 
# 3. All advertising and press materials, printed or electronic, mentioning
# features or use of this software must display the following acknowledgement: 
# 
# 	"This product includes software developed by the Rocks(r)
# 	Cluster Group at the San Diego Supercomputer Center at the
# 	University of California, San Diego and its contributors."
# 
# 4. Except as permitted for the purposes of acknowledgment in paragraph 3,
# neither the name or logo of this software nor the names of its
# authors may be used to endorse or promote products derived from this
# software without specific prior written permission.  The name of the
# software includes the following terms, and any derivatives thereof:
# "Rocks", "Rocks Clusters", and "Avalanche Installer".  For licensing of 
# the associated name, interested parties should contact Technology 
# Transfer & Intellectual Property Services, University of California, 
# San Diego, 9500 Gilman Drive, Mail Code 0910, La Jolla, CA 92093-0910, 
# Ph: (858) 534-5815, FAX: (858) 534-7345, E-MAIL:invent@ucsd.edu
# 
# THIS SOFTWARE IS PROVIDED BY THE REGENTS AND CONTRIBUTORS ``AS IS''
# AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO,
# THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR
# PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE REGENTS OR CONTRIBUTORS
# BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
# CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
# SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR
# BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY,
# WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE
# OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN
# IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
# 
# @Copyright@
#

REDHAT.ROOT = $(PWD)/../../
RPM.EXTRAS=%define __os_install_post /usr/lib/rpm/brp-compress

-include $(ROCKSROOT)/etc/Rules.mk
include $(NBCRDEVEL)/NBCR.mk

COMPILE_FLAGS = -O3 -mtune=core2 -march=core2 -mfpmath=sse -msse3 -fPIC
COMPILE_FFLAGS = -ffree-line-length-none
WRAPPER_FLAGS = -O3
ifeq ("$(COMPILERNAME)", "intel")
  COMPILE_FLAGS = -O3 -fPIC
  COMPILE_FFLAGS = 
else ifeq ("$(COMPILERNAME)", "pgi")
  COMPILE_FLAGS = -fPIC -noswitcherror
  COMPILE_FFLAGS = 
endif

ifneq ("$(ROLLOPTS)", "$(subst avx,,$(ROLLOPTS))")
  ifeq ("$(COMPILERNAME)", "gnu")
    COMPILE_FLAGS += -mavx
  endif
endif

# Removed --enable-mpi-thread-multiple, which can cause hanging processes
# See http://comments.gmane.org/gmane.comp.clustering.open-mpi.user/20338
CONFIG_OPTS = \
  --disable-dlopen --enable-static --enable-mca-static \
  --with-mpi-f90-size=medium \
  --with-platform=optimized --with-wrapper-cflags='$(WRAPPER_FLAGS)' \
  --with-wrapper-cxxflags='$(WRAPPER_FLAGS)' \
  --with-wrapper-fflags='$(WRAPPER_FLAGS)' \
  --with-wrapper-fcflags='$(WRAPPER_FLAGS)' \
  --with-knem=$(KNEM_CACHE)

ifneq ("$(ROLLNETWORK)", "ib")
  CONFIG_OPTS += --without-openib
endif

ifeq ("$(ROLLNETWORK)", "mx")
  MXHOME = /opt/mx
  COMPILE_FLAGS += -I$(MXHOME)/include
  CONFIG_OPTS += --with-mx=$(MXHOME)
endif

ifneq ("$(ROLLOPTS)", "$(subst sge,,$(ROLLOPTS))")
  CONFIG_OPTS += --with-sge
endif

ifneq ("$(ROLLOPTS)", "$(subst torque,,$(ROLLOPTS))")
  CONFIG_OPTS += --with-tm=/opt/torque
endif

KNEM_CACHE = $(REDHAT.ROOT)/cache/build-knem
KNEM_LICENSE_FILES = knem.spec

$(NAME).spec: DESCRIPTION LICENSE

DESCRIPTION:
	echo $(NAME) $(VERSION) > $@

LICENSE: $(SRC_DIRS)
	for F in $(KNEM_LICENSE_FILES); do \
	  cp knem*/$$F $(SOURCE_DIR)/knem-$$F; \
	done
	$(CHECK_LICENSE_FILES)
	echo BSD > $@

build: $(SRC_DIRS)
	( \
	  $(MODULE_LOAD_CC); \
	  export CC=$(CC); \
	  export CFLAGS="$(COMPILE_FLAGS)"; \
	  export CXX=$(CXX); \
	  export CXXFLAGS="$(COMPILE_FLAGS)"; \
	  export F77=$(F77); \
	  export FFLAGS="$(COMPILE_FLAGS) $(COMPILE_FFLAGS)"; \
	  export FC=$(FC); \
	  export FCFLAGS="$(COMPILE_FLAGS) $(COMPILE_FFLAGS)"; \
	  cd $(KNEM_DIR); \
	  ./configure --prefix=$(PKGROOT); \
	  make -j 16; \
	  rm -fr $(KNEM_CACHE); \
	  mkdir -p -m 755 $(KNEM_CACHE); \
	  make prefix=$(KNEM_CACHE) install; \
	  cd ..; \
	  cd $(SOURCE_DIR); \
	  ./configure --prefix=$(PKGROOT) $(CONFIG_OPTS); \
	  make -j 16 \
	)

install::
	mkdir -p -m 755 $(ROOT)/$(PKGROOT)
	$(PKGROOT_BIND_MOUNT)
	( \
	  $(MODULE_LOAD_CC); \
	  cd $(KNEM_DIR); \
	  make prefix=$(PKGROOT) install; \
	  cp `find . -name $(KNEM_KO_NAME)` $(ROOT)/$(PKGROOT)/lib/; \
	  cd ..; \
	  cd $(SOURCE_DIR); \
	  make -j 16 install \
	)
	$(PKGROOT_BIND_UMOUNT)
	$(INSTALL_LICENSE_FILES)

clean::
	rm -fr $(SOURCE_DIR) $(KNEM_DIR) DESCRIPTION LICENSE
