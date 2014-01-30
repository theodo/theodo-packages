NAME    ?= elasticsearch
VERSION ?= 0.90.10# update the SHASUM aswell
VENDOR  ?= elasticsearch
ARCH    ?= x86_64
OS      ?= linux
PREFIX  ?= /etc/elasticsearch

TARDIR     ?= $(NAME)-$(VERSION)
TARBALL    ?= $(TARDIR).tar.gz
INSTALLDIR ?= $(TARDIR)-install
DOWNLOAD   ?= https://download.elasticsearch.org/elasticsearch/elasticsearch/$(TARBALL)
SHASUM     ?= 66db55382bde6085fc9f73225f13649e1f565695

PACKAGE_NAME        ?= $(NAME)
PACKAGE_VERSION     ?= $(VERSION)
PACKAGE_MAINTAINER  ?=# Fill me
PACKAGE_SUFFIX      ?=# Fill me (or not)
PACKAGE_REVISION    ?= 0# the revision should be incremented everytime a change is made, except when VERSION is upgraded then it should be restarted
PACKAGE_EPOCH       ?= 0# should never be used unless a new release is made with an older version of upstream
PACKAGE_URL         ?= http://www.elasticsearch.org/
PACKAGE_CATEGORY    ?= database
define PACKAGE_DESCRIPTION
Distributed restful search and analytics.
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

packages: build-deb build-rpm

build-%: $(TARDIR)
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
		-C"$(TARDIR)" \
		--prefix="$(PREFIX)" \
		--url="$(PACKAGE_URL)" \
		--depends="java7-server-jre" \
		--no-auto-depends \
		--provides="elasticsearch" \
		--package-name-suffix="$(PACKAGE_SUFFIX)" \
		--description="$$PACKAGE_DESCRIPTION" \
		--rpm-os="$(OS)" \
		--rpm-auto-add-directories \
		--no-rpm-sign \
		.
