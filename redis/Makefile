NAME    ?= redis
VERSION ?= 2.8.4# update the SHASUM aswell
VENDOR  ?= Salvatore Sanfilippo
ARCH    ?= x86_64
OS      ?= linux
PREFIX  ?= /usr

TARDIR     ?= $(NAME)-$(VERSION)
TARBALL    ?= $(TARDIR).tar.gz
INSTALLDIR ?= $(TARDIR)-install
DOWNLOAD   ?= http://download.redis.io/releases/$(TARBALL)
SHASUM     ?= 79b156cc9d8bff5a022fdc6d28d5c42999ddcb4f

PACKAGE_NAME        ?= $(NAME)
PACKAGE_VERSION     ?= $(VERSION)
PACKAGE_MAINTAINER  ?=# Fill me
PACKAGE_SUFFIX      ?=# Fill me (or not)
PACKAGE_REVISION    ?= 0# the revision should be incremented everytime a change is made, except when VERSION is upgraded then it should be restarted
PACKAGE_EPOCH       ?= 0# should never be used unless a new release is made with an older version of upstream
PACKAGE_URL         ?= http://www.redis.io/
PACKAGE_CATEGORY    ?= web
define PACKAGE_DESCRIPTION
Redis is an open source, BSD licensed, advanced key-value store. It is often referred to as a data structure server since keys can contain strings, hashes, lists, sets and sorted sets.
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
	cd $(TARDIR)/src \
	&& \
	make install PREFIX=../../$(INSTALLDIR)

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
		-C"$(INSTALLDIR)" \
		--prefix="$(PREFIX)" \
		--url="$(PACKAGE_URL)" \
		--no-depends \
		--no-auto-depends \
		--provides="redis-server" \
		--provides="redis-cli" \
		--provides="redis-benchmark" \
		--provides="redis-check-aof" \
		--provides="redis-check-dump" \
		--package-name-suffix="$(PACKAGE_SUFFIX)" \
		--description="$$PACKAGE_DESCRIPTION" \
		--rpm-os="$(OS)" \
		--rpm-auto-add-directories \
		--no-rpm-sign \
		.
