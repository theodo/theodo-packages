NAME            ?= openresty
VERSION         ?= 1.5.8.1# update the MD5SUM aswell
VENDOR          ?= Yichun Zhang (agentzh)
ARCH            ?= x86_64
CPU             ?= x64
OS              ?= linux
PREFIX          ?= /usr

TARDIR     ?= ngx_$(NAME)-$(VERSION)
TARBALL    ?= $(TARDIR).tar.gz
INSTALLDIR ?= $(TARDIR)-install
DOWNLOAD   ?= http://openresty.org/download/$(TARBALL)
MD5SUM     ?= a80d35af95e1dc20c97a7a0df81aeb0d

PACKAGE_NAME        ?= $(NAME)
PACKAGE_VERSION     ?= $(VERSION)
PACKAGE_MAINTAINER  ?=# Fill me
PACKAGE_PREFIX      ?= /
PACKAGE_SUFFIX      ?=# Fill me (or not)
PACKAGE_REVISION    ?= 0# the revision should be incremented everytime a change is made, except when VERSION is upgraded then it should be restarted
PACKAGE_EPOCH       ?= 0# should never be used unless a new release is made with an older version of upstream
PACKAGE_URL         ?= http://www.openresty.org/
PACKAGE_CATEGORY    ?= web
define PACKAGE_DESCRIPTION
OpenResty (aka. ngx_openresty) is a full-fledged web application server by bundling the standard Nginx core, lots of 3rd-party Nginx modules, as well as most of their external dependencies.
endef
export PACKAGE_DESCRIPTION

define AFTER_INSTALL_SCRIPT
#!/bin/sh

set -e

PREFIX=$(PACKAGE_PREFIX)

if [ -n $$RPM_INSTALL_PREFIX ] ; then
  PREFIX=$$RPM_INSTALL_PREFIX
fi

cat > $(PACKAGE_PREFIX)/etc/$(NAME)/$(NAME).conf << EOF
worker_processes  4;

error_log $$PREFIX/var/log/openresty/error.log;

pid       $$PREFIX/var/run/nginx.pid;

events {
    worker_connections  2048;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    access_log    $$PREFIX/var/log/openresty/access.log main;

    sendfile        on;
    tcp_nopush     on;

    keepalive_timeout  65;

    gzip  on;
    gzip_http_version 1.1;
    gzip_vary on;
    gzip_comp_level 9;
    gzip_proxied any;
    gzip_types text/plain text/html text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript application/javascript;
    gzip_buffers 16 8k;

    include conf.d/*.conf;

    server {
        listen       80;
        server_name  localhost;

        location / {
            root   html;
            index  index.html index.htm;
        }
    }

    include sites-enabled/*;
}
EOF

endef
export AFTER_INSTALL_SCRIPT

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
	export PATH=/sbin:$$PATH \
	&& \
	./configure \
		--prefix=$(PREFIX) \
		--with-luajit \
		--with-http_postgres_module \
		--with-libpq=/usr/pgsql-9.3 \
		--sbin-path=/usr/bin/openresty \
		--conf-path=/etc/openresty/openresty.conf \
		--http-log-path=/var/log/openresty \
		--error-log-path=/var/log/openresty \
	&& \
	make \
	&& \
	make install DESTDIR=`pwd`/../$(INSTALLDIR)

install: build-source
	mkdir -p $(INSTALLDIR)/etc/$(NAME)/conf.d
	mkdir -p $(INSTALLDIR)/etc/$(NAME)/sites-available
	mkdir -p $(INSTALLDIR)/etc/$(NAME)/sites-enabled
	mkdir -p $(INSTALLDIR)/var/log/$(NAME)
	rm -rf $(INSTALLDIR)$(PREFIX)/nginx
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
		-C"$(INSTALLDIR)$(PACKAGE_PREFIX)" \
		--prefix="$(PACKAGE_PREFIX)" \
		--url="$(PACKAGE_URL)" \
		--no-depends \
		--no-auto-depends \
		--provides="nginx" \
		--after-install="after-install.sh" \
		--package-name-suffix="$(PACKAGE_SUFFIX)" \
		--description="$$PACKAGE_DESCRIPTION" \
		--rpm-os="$(OS)" \
		--rpm-auto-add-directories \
		--no-rpm-sign \
		.
