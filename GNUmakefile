# GNUmakefile
# 
# Copyright 2008 Bryan Ischo <bryan@ischo.com>
# 
# This file is part of libs3.
# 
# libs3 is free software: you can redistribute it and/or modify it under the
# terms of the GNU General Public License as published by the Free Software
# Foundation, version 3 of the License.
#
# In addition, as a special exception, the copyright holders give
# permission to link the code of this library and its programs with the
# OpenSSL library, and distribute linked combinations including the two.
#
# libs3 is distributed in the hope that it will be useful, but WITHOUT ANY
# WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS
# FOR A PARTICULAR PURPOSE.  See the GNU General Public License for more
# details.
#
# You should have received a copy of the GNU General Public License version 3
# along with libs3, in a file named COPYING.  If not, see
# <http://www.gnu.org/licenses/>.

# I tried to use the autoconf/automake/autolocal/etc (i.e. autohell) tools
# but I just couldn't stomach them.  Since this is a Makefile for POSIX
# systems, I will simply do away with autohell completely and use a GNU
# Makefile.  GNU make ought to be available pretty much everywhere, so I
# don't see this being a significant issue for portability.

# All commands assume a GNU compiler.  For systems which do not use a GNU
# compiler, write scripts with the same names as these commands, and taking
# the same arguments, and translate the arguments and commands into the
# appropriate non-POSIX ones as needed.  libs3 assumes a GNU toolchain as
# the most portable way to build software possible.  Non-POSIX, non-GNU
# systems can do the work of supporting this build infrastructure.


# --------------------------------------------------------------------------
# Set libs3 version number

LIBS3_VER_MAJOR := 1
LIBS3_VER_MINOR := 2
LIBS3_VER := $(LIBS3_VER_MAJOR).$(LIBS3_VER_MINOR)


# --------------------------------------------------------------------------
# BUILD directory
ifndef BUILD
    BUILD := build
endif


# --------------------------------------------------------------------------
# DESTDIR directory
ifndef DESTDIR
    DESTDIR := /usr
endif


# --------------------------------------------------------------------------
# Acquire configuration information for libraries that libs3 depends upon

ifndef CURL_LIBS
    CURL_LIBS := $(shell curl-config --libs)
endif

ifndef CURL_CFLAGS
    CURL_CFLAGS := $(shell curl-config --cflags)
endif

ifndef LIBXML2_LIBS
    LIBXML2_LIBS := $(shell xml2-config --libs)
endif

ifndef LIBXML2_CFLAGS
    LIBXML2_CFLAGS := $(shell xml2-config --cflags)
endif


# --------------------------------------------------------------------------
# These CFLAGS assume a GNU compiler.  For other compilers, write a script
# which converts these arguments into their equivalent for that particular
# compiler.

ifndef CFLAGS
    CFLAGS = -O3
endif

CFLAGS += -Wall -Werror -Wshadow -Wextra -std=c99 -Iinc \
          $(CURL_CFLAGS) $(LIBXML2_CFLAGS) \
          -DLIBS3_VER_MAJOR=\"$(LIBS3_VER_MAJOR)\" \
          -DLIBS3_VER_MINOR=\"$(LIBS3_VER_MINOR)\" \
          -DLIBS3_VER=\"$(LIBS3_VER)\"

LDFLAGS = $(CURL_LIBS) $(LIBXML2_LIBS) -lpthread


# --------------------------------------------------------------------------
# Default targets are everything

.PHONY: all
all: exported test


# --------------------------------------------------------------------------
# Exported targets are the library and driver program

.PHONY: exported
exported: libs3 s3 headers


# --------------------------------------------------------------------------
# Install target

.PHONY: install
install: libs3 s3 headers
	install -Dps -m u+rwx,go+rx $(BUILD)/bin/s3 $(DESTDIR)/bin/s3
	install -Dp -m u+rw,go+r $(BUILD)/include/libs3.h \
               $(DESTDIR)/include/libs3.h
	install -Dp -m u+rw,go+r $(BUILD)/lib/libs3.a $(DESTDIR)/lib/libs3.a
	install -Dps -m u+rw,go+r $(BUILD)/lib/libs3.so.$(LIBS3_VER_MAJOR) \
               $(DESTDIR)/lib/libs3.so.$(LIBS3_VER)
	ln -sf libs3.so.$(LIBS3_VER) $(DESTDIR)/lib/libs3.so.$(LIBS3_VER_MAJOR)
	ln -sf libs3.so.$(LIBS3_VER_MAJOR) $(DESTDIR)/lib/libs3.so


# --------------------------------------------------------------------------
# Uninstall target

.PHONY: uninstall
uninstall:
	rm -f $(DESTDIR)/bin/s3 \
              $(DESTDIR)/include/libs3.h \
              $(DESTDIR)/lib/libs3.a \
              $(DESTDIR)/lib/libs3.so \
              $(DESTDIR)/lib/libs3.so.$(LIBS3_VER_MAJOR) \
              $(DESTDIR)/lib/libs3.so.$(LIBS3_VER) \


# --------------------------------------------------------------------------
# Debian package target

DEBPKG = $(BUILD)/pkg/libs3_$(LIBS3_VER).deb
DEBDEVPKG = $(BUILD)/pkg/libs3-dev_$(LIBS3_VER).deb

.PHONY: deb
deb: $(DEBPKG) $(DEBDEVPKG)

$(DEBPKG): DEBARCH = $(shell dpkg-architecture | grep ^DEB_BUILD_ARCH= | \
                       cut -d '=' -f 2)
$(DEBPKG): exported $(BUILD)/deb/DEBIAN/control $(BUILD)/deb/DEBIAN/shlibs \
           $(BUILD)/deb/DEBIAN/postinst \
           $(BUILD)/deb/usr/share/doc/libs3/changelog.gz \
           $(BUILD)/deb/usr/share/doc/libs3/changelog.Debian.gz \
           $(BUILD)/deb/usr/share/doc/libs3/copyright
	DESTDIR=$(BUILD)/deb/usr $(MAKE) install
	rm -rf $(BUILD)/deb/usr/include
	rm -f $(BUILD)/deb/usr/lib/libs3.a
	@mkdir -p $(dir $@)
	fakeroot dpkg-deb -b $(BUILD)/deb $@
	mv $@ $(BUILD)/pkg/libs3_$(LIBS3_VER)_$(DEBARCH).deb

$(DEBDEVPKG): DEBARCH = $(shell dpkg-architecture | grep ^DEB_BUILD_ARCH= | \
                          cut -d '=' -f 2)
$(DEBDEVPKG): exported $(BUILD)/deb-dev/DEBIAN/control \
           $(BUILD)/deb-dev/usr/share/doc/libs3-dev/changelog.gz \
           $(BUILD)/deb-dev/usr/share/doc/libs3-dev/changelog.Debian.gz \
           $(BUILD)/deb-dev/usr/share/doc/libs3-dev/copyright
	DESTDIR=$(BUILD)/deb-dev/usr $(MAKE) install
	rm -rf $(BUILD)/deb-dev/usr/bin
	rm -f $(BUILD)/deb-dev/usr/lib/libs3.so*
	@mkdir -p $(dir $@)
	fakeroot dpkg-deb -b $(BUILD)/deb-dev $@
	mv $@ $(BUILD)/pkg/libs3-dev_$(LIBS3_VER)_$(DEBARCH).deb

$(BUILD)/deb/DEBIAN/control: debian/control
	@mkdir -p $(dir $@)
	echo -n "Depends: " > $@
	dpkg-shlibdeps -O $(BUILD)/lib/libs3.so.$(LIBS3_VER_MAJOR) | \
            cut -d '=' -f 2- >> $@
	sed -e 's/LIBS3_VERSION/$(LIBS3_VER)/' \
            < $< | sed -e 's/DEBIAN_ARCHITECTURE/$(DEBARCH)/' | \
            grep -v ^Source: >> $@

$(BUILD)/deb-dev/DEBIAN/control: debian/control.dev
	@mkdir -p $(dir $@)
	sed -e 's/LIBS3_VERSION/$(LIBS3_VER)/' \
            < $< | sed -e 's/DEBIAN_ARCHITECTURE/$(DEBARCH)/' >> $@

$(BUILD)/deb/DEBIAN/shlibs:
	echo -n "libs3 $(LIBS3_VER_MAJOR) libs3 " > $@
	echo "(>= $(LIBS3_VER))" >> $@

$(BUILD)/deb/DEBIAN/postinst: debian/postinst
	@mkdir -p $(dir $@)
	cp $< $@

$(BUILD)/deb/usr/share/doc/libs3/copyright: LICENSE
	@mkdir -p $(dir $@)
	cp $< $@
	@echo >> $@
	@echo -n "An alternate location for the GNU General Public " >> $@
	@echo "License version 3 on Debian" >> $@
	@echo "systems is /usr/share/common-licenses/GPL-3." >> $@

$(BUILD)/deb-dev/usr/share/doc/libs3-dev/copyright: LICENSE
	@mkdir -p $(dir $@)
	cp $< $@
	@echo >> $@
	@echo -n "An alternate location for the GNU General Public " >> $@
	@echo "License version 3 on Debian" >> $@
	@echo "systems is /usr/share/common-licenses/GPL-3." >> $@

$(BUILD)/deb/usr/share/doc/libs3/changelog.gz: debian/changelog
	@mkdir -p $(dir $@)
	gzip --best -c $< > $@

$(BUILD)/deb-dev/usr/share/doc/libs3-dev/changelog.gz: debian/changelog
	@mkdir -p $(dir $@)
	gzip --best -c $< > $@

$(BUILD)/deb/usr/share/doc/libs3/changelog.Debian.gz: debian/changelog.Debian
	@mkdir -p $(dir $@)
	gzip --best -c $< > $@

$(BUILD)/deb-dev/usr/share/doc/libs3-dev/changelog.Debian.gz: \
    debian/changelog.Debian
	@mkdir -p $(dir $@)
	gzip --best -c $< > $@


# --------------------------------------------------------------------------
# Compile target patterns

$(BUILD)/obj/%.o: src/%.c
	@mkdir -p $(dir $@)
	gcc $(CFLAGS) -o $@ -c $<

$(BUILD)/obj/%.do: src/%.c
	@mkdir -p $(dir $@)
	gcc $(CFLAGS) -fpic -fPIC -o $@ -c $< 


# --------------------------------------------------------------------------
# libs3 library targets

LIBS3_SHARED = $(BUILD)/lib/libs3.so.$(LIBS3_VER_MAJOR)

.PHONY: libs3
libs3: $(LIBS3_SHARED) $(LIBS3_SHARED_MAJOR) $(BUILD)/lib/libs3.a

LIBS3_SOURCES := src/acl.c src/bucket.c src/error_parser.c src/general.c \
                 src/object.c src/request.c src/request_context.c \
                 src/response_headers_handler.c src/service_access_logging.c \
                 src/service.c src/simplexml.c src/util.c

$(LIBS3_SHARED): $(LIBS3_SOURCES:src/%.c=$(BUILD)/obj/%.do)
	@mkdir -p $(dir $@)
	gcc -shared -Wl,-soname,libs3.so.$(LIBS3_VER_MAJOR) -o $@ $^ \
            $(S3_LIBS) $(LDFLAGS)

$(BUILD)/lib/libs3.a: $(LIBS3_SOURCES:src/%.c=$(BUILD)/obj/%.o)
	@mkdir -p $(dir $@)
	$(AR) cr $@ $^


# --------------------------------------------------------------------------
# Driver program targets

.PHONY: s3
s3: $(BUILD)/bin/s3

$(BUILD)/bin/s3: $(BUILD)/obj/s3.o $(LIBS3_SHARED)
	@mkdir -p $(dir $@)
	gcc -o $@ $^ $(LDFLAGS)


# --------------------------------------------------------------------------
# libs3 header targets

.PHONY: headers
headers: $(BUILD)/include/libs3.h

$(BUILD)/include/libs3.h: inc/libs3.h
	@mkdir -p $(dir $@)
	cp $< $@


# --------------------------------------------------------------------------
# Test targets

.PHONY: test
test: $(BUILD)/bin/testsimplexml

$(BUILD)/bin/testsimplexml: $(BUILD)/obj/testsimplexml.o $(BUILD)/lib/libs3.a
	@mkdir -p $(dir $@)
	gcc -o $@ $^ $(LIBXML2_LIBS)


# --------------------------------------------------------------------------
# Clean target

.PHONY: clean
clean:
	rm -rf $(BUILD)
