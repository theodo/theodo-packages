NAME            ?= java7-server-jre
VERSION         ?= 7u51# update the BUILD, NUMERIC_VERSION and MD5SUM aswell
BUILD           ?= b13# download java manually to get the build number
NUMERIC_VERSION ?= 1.7.0_51
VENDOR          ?= Oracle
ARCH            ?= x86_64
CPU             ?= x64
OS              ?= linux
PREFIX          ?= /usr

TARDIR     ?= jdk$(NUMERIC_VERSION)
TARBALL    ?= $(VERSION)-$(OS)-$(CPU).tar.gz
INSTALLDIR ?= $(TARDIR)-install
JAVADIR    ?= /lib/java/$(NAME)
DESTDIR    ?= $(INSTALLDIR)$(PREFIX)$(JAVADIR)
DOWNLOAD   ?= http://download.oracle.com/otn-pub/java/jdk/$(VERSION)-$(BUILD)/server-jre-$(TARBALL)
MD5SUM     ?= c5a034f4222bac326101799bcb20509c

PACKAGE_NAME        ?= $(NAME)
PACKAGE_VERSION     ?= $(VERSION)
PACKAGE_MAINTAINER  ?=# Fill me
PACKAGE_SUFFIX      ?=# Fill me (or not)
PACKAGE_REVISION    ?= 0# the revision should be incremented everytime a change is made, except when VERSION is upgraded then it should be restarted
PACKAGE_EPOCH       ?= 0# should never be used unless a new release is made with an older version of upstream
PACKAGE_URL         ?= http://www.java.com/
PACKAGE_CATEGORY    ?= devel
define PACKAGE_DESCRIPTION
Java is a programming language and computing platform first released by Sun Microsystems in 1995.
endef
export PACKAGE_DESCRIPTION


.PHONY: all

all: packages

clean:
	rm -rf $(TARDIR)
	rm -rf $(INSTALLDIR)

distclean: clean
	rm -rf $(TARBALL)
	rm -f *.rpm *.deb

$(TARBALL):
	echo -n "$(MD5SUM)  $(TARBALL)"|md5sum -c || wget --no-check-certificate --no-cookies --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com" $(DOWNLOAD) -O $(TARBALL)

$(TARDIR): $(TARBALL)
	tar xvzf $(TARBALL)

packages: build-deb build-rpm

install: $(TARDIR)
	mkdir -p $(DESTDIR)
	mkdir -p $(INSTALLDIR)$(PREFIX)/bin
	cp -r $(TARDIR)/* $(DESTDIR)
	ln -s ../$(JAVADIR)/bin/java $(INSTALLDIR)$(PREFIX)/bin/java
	ln -s ../$(JAVADIR)/bin/javac $(INSTALLDIR)$(PREFIX)/bin/javac

build-%: install
	fpm \
		--force \
		-sdir \
		-t$* \
		--version="$(PACKAGE_VERSION)" \
		--iteration="$(PACKAGE_REVISION)" \
		--epoch="$(PACKAGE_EPOCH)" \
		--vendor="$(VENDOR)" \
		--name="$(PACKAGE_NAME)" \
		--architecture="$(ARCH)" \
		--category="$(PACKAGE_CATEGORY)" \
		-C"$(INSTALLDIR)$(PREFIX)" \
		--prefix="$(PREFIX)" \
		--url="$(PACKAGE_URL)" \
		--no-depends \
		--no-auto-depends \
		--provides="java" \
		--provides="javac" \
		--package-name-suffix="$(PACKAGE_SUFFIX)" \
		--description="$$PACKAGE_DESCRIPTION" \
		--rpm-os="$(OS)" \
		--rpm-auto-add-directories \
		--no-rpm-sign \
		.
