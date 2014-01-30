NAME    ?= node
VERSION ?= 0.10.24# update the SHASUM aswell
VENDOR  ?= joyent
CPU     ?= x64
ARCH    ?= x86_64
OS      ?= linux
PREFIX  ?= /usr

TARDIR     ?= $(NAME)-v$(VERSION)
TARBALL    ?= $(TARDIR).tar.gz
INSTALLDIR ?= $(TARDIR)-install
DOWNLOAD   ?= http://nodejs.org/dist/v$(VERSION)/$(TARBALL)
SHASUM     ?= d162d01eb173cb5a0e7e46c9d706421c4c771039

PACKAGE_NAME        ?= $(NAME)
PACKAGE_VERSION     ?= $(VERSION)
PACKAGE_MAINTAINER  ?=# Fill me
PACKAGE_SUFFIX      ?=# Fill me (or not)
PACKAGE_REVISION    ?= 0# the revision should be incremented everytime a change is made, except when VERSION is upgraded then it should be restarted
PACKAGE_EPOCH       ?= 0# should never be used unless a new release is made with an older version of upstream
PACKAGE_URL         ?= http://www.nodejs.org/
PACKAGE_CATEGORY    ?= web
define PACKAGE_DESCRIPTION
Node.js is a platform built on Chrome's JavaScript runtime for easily building fast, scalable network applications. Node.js uses an event-driven, non-blocking I/O model that makes it lightweight and efficient, perfect for data-intensive real-time applications that run across distributed devices.

Node.js is bundled with several useful libraries to handle server tasks:

System, Events, Standard I/O, Modules, Timers, Child Processes, POSIX, HTTP, Multipart Parsing, TCP, DNS, Assert, Path, URL, Query Strings.
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
	echo -n "$(SHASUM)  $(TARBALL)"|shasum -c || wget -N $(DOWNLOAD)

$(TARDIR): $(TARBALL)
	tar xvzf $(TARBALL)

build-source: $(TARDIR)
	export \
		PROCESSOR_ARCHITECTURE=$(CPU) \
		PORTABLE \
	&& \
	cd $(TARDIR) \
	&& \
	./configure \
		--prefix=/usr \
		--dest-os=$(OS) \
		--dest-cpu=$(CPU) \
		--no-ssl2 \
		--without-perfctr \
		--without-etw \
		--without-dtrace \
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
		--version=$(PACKAGE_VERSION) \
		--iteration=$(PACKAGE_REVISION) \
		--epoch=$(PACKAGE_EPOCH) \
		--vendor=$(VENDOR) \
		--name=$(PACKAGE_NAME) \
		--architecture=$(ARCH) \
		--category=$(PACKAGE_CATEGORY) \
		-C$(INSTALLDIR)$(PREFIX) \
		--prefix $(PREFIX) \
		--url=$(PACKAGE_URL) \
		--no-depends \
		--no-auto-depends \
		--provides=node \
		--provides=npm \
		--package-name-suffix=$(PACKAGE_SUFFIX) \
		--description="$$PACKAGE_DESCRIPTION" \
		--rpm-os=$(OS) \
		--rpm-auto-add-directories \
		--no-rpm-sign \
		.
