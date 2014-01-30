NAME            ?= python2
VERSION         ?= 2.7.6# update the MD5SUM aswell
VENDOR          ?= Python Software Foundation
ARCH            ?= x86_64
CPU             ?= x64
OS              ?= linux
PREFIX          ?= /usr

TARDIR     ?= Python-$(VERSION)
TARBALL    ?= $(TARDIR).tgz
INSTALLDIR ?= $(TARDIR)-install
DOWNLOAD   ?= http://python.org/ftp/python/$(VERSION)/$(TARBALL)
MD5SUM     ?= 1d8728eb0dfcac72a0fd99c17ec7f386

PACKAGE_NAME        ?= $(NAME)
PACKAGE_VERSION     ?= $(VERSION)
PACKAGE_MAINTAINER  ?=# Fill me
PACKAGE_SUFFIX      ?=# Fill me (or not)
PACKAGE_REVISION    ?= 0# the revision should be incremented everytime a change is made, except when VERSION is upgraded then it should be restarted
PACKAGE_EPOCH       ?= 0# should never be used unless a new release is made with an older version of upstream
PACKAGE_URL         ?= http://www.python.org/
PACKAGE_CATEGORY    ?= devel
define PACKAGE_DESCRIPTION
Python is a programming language that lets you work more quickly and integrate your systems more effectively. You can learn to use Python and see almost immediate gains in productivity and lower maintenance costs.
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
	echo -n "$(MD5SUM)  $(TARBALL)"|md5sum -c || wget -N $(DOWNLOAD)

$(TARDIR): $(TARBALL)
	tar xvzf $(TARBALL)

build-source: $(TARDIR)
	cd $(TARDIR) \
	&& \
	./configure \
		--prefix=$(PREFIX) \
	&& \
	make \
	&& \
	make install DESTDIR=../$(INSTALLDIR)

packages: build-deb build-rpm

build-%: build-source
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
		--provides="python" \
		--provides="python2.7" \
		--package-name-suffix="$(PACKAGE_SUFFIX)" \
		--description="$$PACKAGE_DESCRIPTION" \
		--rpm-os="$(OS)" \
		--rpm-auto-add-directories \
		--no-rpm-sign \
		.
