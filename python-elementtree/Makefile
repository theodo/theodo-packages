NAME            ?= elementtree
VERSION         ?= 1.2.6-20050316# update the MD5SUM aswell
VENDOR          ?= Effbot
ARCH            ?= x86_64
CPU             ?= x64
OS              ?= linux
PREFIX          ?= /usr

TARDIR     ?= $(NAME)-$(VERSION)
TARBALL    ?= $(TARDIR).tar.gz
INSTALLDIR ?= install
SRCRELDIR  ?= src/python
DESTDIR    ?= $(INSTALLDIR)$(PREFIX)/$(SRCRELDIR)
DOWNLOAD   ?= http://effbot.org/media/downloads/$(TARBALL)
MD5SUM     ?= e1b21716be8bfff8bf192fc3880ad008

PACKAGE_NAME        ?= python-$(NAME)
PACKAGE_VERSION     ?= $(VERSION)
PACKAGE_MAINTAINER  ?=# Fill me
PACKAGE_SUFFIX      ?=# Fill me (or not)
PACKAGE_REVISION    ?= 0# the revision should be incremented everytime a change is made, except when VERSION is upgraded then it should be restarted
PACKAGE_EPOCH       ?= 0# should never be used unless a new release is made with an older version of upstream
PACKAGE_URL         ?= https://pypi.python.org/pypi/elementtree
PACKAGE_CATEGORY    ?= devel
define PACKAGE_DESCRIPTION
The Element type is a flexible container object, designed to store hierarchical data structures in memory. Element structures can be converted to and from XML.
endef
export PACKAGE_DESCRIPTION

define AFTER_INSTALL_SCRIPT
#!/bin/sh

set -e

PREFIX=$(PREFIX)

if [ -n $$RPM_INSTALL_PREFIX ] ; then
  PREFIX=$$RPM_INSTALL_PREFIX
fi

cd $$PREFIX/$(SRCRELDIR)/$(TARDIR) && $$PREFIX/bin/python setup.py install
endef
export AFTER_INSTALL_SCRIPT


.PHONY: all

all: packages

clean:
	rm -rf $(TARDIR)
	rm -rf $(INSTALLDIR)
	rm -rf *.sh

distclean: clean
	rm -rf $(TARBALL)
	rm -f *.rpm *.deb

$(TARBALL):
	echo -n "$(MD5SUM)  $(TARBALL)"|md5sum -c || wget --no-check-certificate -N $(DOWNLOAD)

$(TARDIR): $(TARBALL)
	mkdir -p $(TARDIR)
	tar xvzf $(TARBALL) -C $(TARDIR)/..

install: $(TARDIR)
	mkdir -p $(DESTDIR)
	cp -r $(TARDIR) $(DESTDIR)
	echo "$$AFTER_INSTALL_SCRIPT" > after-install.sh

packages: build-deb build-rpm

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
		--depends="python2 >= 2.7" \
		--no-auto-depends \
		--after-install="after-install.sh" \
		--package-name-suffix="$(PACKAGE_SUFFIX)" \
		--description="$$PACKAGE_DESCRIPTION" \
		--rpm-os="$(OS)" \
		--rpm-auto-add-directories \
		--no-rpm-sign \
		.
