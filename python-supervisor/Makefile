NAME            ?= supervisor
VERSION         ?= 3.0# update the MD5SUM aswell
VENDOR          ?= Agendaless Consulting
ARCH            ?= x86_64
CPU             ?= x64
OS              ?= linux
PREFIX          ?= /

TARDIR     ?= $(NAME)-$(VERSION)
TARBALL    ?= $(TARDIR).tar.gz
INSTALLDIR ?= install
SRCRELDIR  ?= usr/src/python
DESTDIR    ?= $(INSTALLDIR)$(PREFIX)/$(SRCRELDIR)
DOWNLOAD   ?= https://pypi.python.org/packages/source/s/$(NAME)/$(TARBALL)
MD5SUM     ?= 94ff3cf09618c36889425a8e002cd51a

PACKAGE_NAME        ?= python-$(NAME)
PACKAGE_VERSION     ?= $(VERSION)
PACKAGE_MAINTAINER  ?=# Fill me
PACKAGE_SUFFIX      ?=# Fill me (or not)
PACKAGE_REVISION    ?= 3# the revision should be incremented everytime a change is made, except when VERSION is upgraded then it should be restarted
PACKAGE_EPOCH       ?= 0# should never be used unless a new release is made with an older version of upstream
PACKAGE_URL         ?= https://pypi.python.org/pypi/supervisor
PACKAGE_CATEGORY    ?= devel
define PACKAGE_DESCRIPTION
Supervisor is a client/server system that allows its users to control a number of processes on UNIX-like operating systems.
endef
export PACKAGE_DESCRIPTION

define AFTER_INSTALL_SCRIPT
#!/bin/sh

set -e

PREFIX=$(PREFIX)

if [ -n $$RPM_INSTALL_PREFIX ] ; then
  PREFIX=$$RPM_INSTALL_PREFIX
fi

cd $$PREFIX/$(SRCRELDIR)/$(TARDIR) && $$PREFIX/usr/bin/python setup.py install

cat > $$PREFIX/etc/$(NAME)/$(NAME).conf <<EOF
; supervisor config file

[unix_http_server]
file=$$PREFIX/var/run/$(NAME).sock ; (the path to the socket file)
chmod=0700                         ; sockef file mode (default 0700)

[supervisord]
logfile=$$PREFIX/var/log/$(NAME)/$(NAME).log ; (main log file)
pidfile=$$PREFIX/var/run/$(NAME).pid         ; (supervisord pidfile)
childlogdir=$$PREFIX/var/log/$(NAME)         ; ('AUTO' child log dir, default $TEMP)

; the below section must remain in the config file for RPC
; (supervisorctl/web interface) to work, additional interfaces may be
; added by defining them in separate rpcinterface: sections
[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

[supervisorctl]
serverurl=unix://$$PREFIX/var/run/$(NAME).sock ; use a unix:// URL  for a unix socket

; The [include] section can just contain the "files" setting.  This
; setting can list multiple files (separated by whitespace or
; newlines).  It can also contain wildcards.  The filenames are
; interpreted as relative to this file.  Included files *cannot*
; include files themselves.

[include]
files = $$PREFIX/etc/$(NAME)/conf.d/*.conf
EOF
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
	mkdir -p $(INSTALLDIR)/etc/supervisor/conf.d
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
		--depends="python-setuptools" \
		--depends="python-meld3" \
		--depends="python-elementtree" \
		--config-files="/etc/supervisor/supervisor.conf" \
		--directories="/etc/supervisor" \
		--no-auto-depends \
		--provides="supervisorctl" \
		--provides="supervisord" \
		--after-install="after-install.sh" \
		--package-name-suffix="$(PACKAGE_SUFFIX)" \
		--description="$$PACKAGE_DESCRIPTION" \
		--rpm-os="$(OS)" \
		--rpm-auto-add-directories \
		--no-rpm-sign \
		.
