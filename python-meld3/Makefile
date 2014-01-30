NAME            ?= meld3
VERSION         ?= 0.6.10# update the MD5SUM aswell
VENDOR          ?= Plope
ARCH            ?= x86_64
CPU             ?= x64
OS              ?= linux
PREFIX          ?= /usr

TARDIR     ?= $(NAME)-$(VERSION)
TARBALL    ?= $(TARDIR).tar.gz
INSTALLDIR ?= install
SRCRELDIR  ?= src/python
DESTDIR    ?= $(INSTALLDIR)$(PREFIX)/$(SRCRELDIR)
DOWNLOAD   ?= https://pypi.python.org/packages/source/m/meld3/$(TARBALL)
MD5SUM     ?= 42e58624e9d427be7659d7a28e2b0b6f

PACKAGE_NAME        ?= python-$(NAME)
PACKAGE_VERSION     ?= $(VERSION)
PACKAGE_MAINTAINER  ?=# Fill me
PACKAGE_SUFFIX      ?=# Fill me (or not)
PACKAGE_REVISION    ?= 0# the revision should be incremented everytime a change is made, except when VERSION is upgraded then it should be restarted
PACKAGE_EPOCH       ?= 0# should never be used unless a new release is made with an older version of upstream
PACKAGE_URL         ?= https://pypi.python.org/pypi/meld3
PACKAGE_CATEGORY    ?= devel
define PACKAGE_DESCRIPTION
meld3 is an HTML/XML templating engine.
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
