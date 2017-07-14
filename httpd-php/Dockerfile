FROM alpine:3.6


#
# ENV httpd
#
ENV HTTPD_VERSION 2.4.25
ENV HTTPD_PREFIX /usr/local/apache2
ENV PATH $HTTPD_PREFIX/bin:$PATH

ENV HTTPD_BZ2_MD5 https://archive.apache.org/dist/httpd/httpd-$HTTPD_VERSION.tar.bz2.md5
ENV HTTPD_BZ2_SHA1 https://archive.apache.org/dist/httpd/httpd-$HTTPD_VERSION.tar.bz2.sha1
ENV HTTPD_BZ2_SHA256 https://archive.apache.org/dist/httpd/httpd-$HTTPD_VERSION.tar.bz2.sha256

ENV HTTPD_BZ2_URL https://apache.org/dyn/closer.cgi?action=download&filename=httpd/httpd-$HTTPD_VERSION.tar.bz2
ENV HTTPD_BZ2_URL_ARCHIVE https://archive.apache.org/dist/httpd/httpd-$HTTPD_VERSION.tar.bz2
ENV HTTPD_ASC_URL https://www.apache.org/dist/httpd/httpd-$HTTPD_VERSION.tar.bz2.asc
ENV HTTPD_ASC_URL_ARCHIVE https://archive.apache.org/dist/httpd/httpd-$HTTPD_VERSION.tar.bz2.asc
ENV HTTPD_KEYS_URL http://www.apache.org/dist/httpd/KEYS

#
# ENV php
#
ENV PHP_VERSION 7.1.7
ENV PHP_PREFIX /usr/local/php
ENV PHP_INI_DIR /usr/local/etc/php

# Apply stack smash protection to functions using local buffers and alloca()
# Make PHP's main executable position-independent (improves ASLR security mechanism, and has no performance impact on x86_64)
# Enable optimization (-O2)
# Enable linker optimization (this sorts the hash buckets to improve cache locality, and is non-default)
# Adds GNU HASH segments to generated executables (this is used if present, and is much faster than sysv hash; in this configuration, sysv hash is also generated)
# https://github.com/docker-library/php/issues/272
ENV PHP_CFLAGS="-fstack-protector-strong -fpic -fpie -O2"
ENV PHP_CPPFLAGS="$PHP_CFLAGS"
ENV PHP_LDFLAGS="-Wl,-O1 -Wl,--hash-style=both -pie"

ENV PHP_URL https://secure.php.net/get/php-$PHP_VERSION.tar.xz/from/this/mirror
ENV PHP_ASC_URL https://secure.php.net/get/php-$PHP_VERSION.tar.xz.asc/from/this/mirror
ENV PHP_SHA256 0d42089729be7b2bb0308cbe189c2782f9cb4b07078c8a235495be5874fff729

# php 7.1 Joe Watkins <krakjoe@php.net>
ENV GPG_KEY 31CBD89E


#
# COPY
#
COPY httpd-foreground /usr/local/bin/


#
# RUN
# one run to create one layer
#
RUN \
set -xe; \
mkdir -p $PHP_INI_DIR/conf.d; \
chmod +x /usr/local/bin/httpd-foreground; \


#
# www-data
# 82 is the standard uid/gid for "www-data" in Alpine
#
# http://git.alpinelinux.org/cgit/aports/tree/main/apache2/apache2.pre-install?h=v3.3.2
# http://git.alpinelinux.org/cgit/aports/tree/main/lighttpd/lighttpd.pre-install?h=v3.3.2
# http://git.alpinelinux.org/cgit/aports/tree/main/nginx-initscripts/nginx-initscripts.pre-install?h=v3.3.2
#
# create a system group, www-data, with gid 82
# create a system user, www-data, in the group, www-data, with uid 82, no password, no login shell
addgroup -g 82 -S www-data; \
adduser -u 82 -D -S -s /sbin/nologin -G www-data www-data; \


#
# get distro - shared
#
apk add --no-cache --virtual .wget-deps \
ca-certificates \
wget \
; \


#
# get distro - httpd
#
mkdir -p "$HTTPD_PREFIX"; \
chown www-data:www-data "$HTTPD_PREFIX"; \
cd "$HTTPD_PREFIX"; \

if ! wget -O httpd-$HTTPD_VERSION.tar.bz2 "$HTTPD_BZ2_URL"; then \
wget -O httpd-$HTTPD_VERSION.tar.bz2 "$HTTPD_BZ2_URL_ARCHIVE"; \
wget -O httpd-$HTTPD_VERSION.tar.bz2.asc "$HTTPD_ASC_URL_ARCHIVE"; \
else \
wget -O httpd-$HTTPD_VERSION.tar.bz2.asc "$HTTPD_ASC_URL"; \
fi; \

wget -O httpd-$HTTPD_VERSION.tar.bz2.md5 "$HTTPD_BZ2_MD5"; \
wget -O httpd-$HTTPD_VERSION.tar.bz2.sha1 "$HTTPD_BZ2_SHA1"; \

if ! wget -O httpd-$HTTPD_VERSION.tar.bz2.sha256 "$HTTPD_BZ2_SHA256"; then \
echo 'no SHA256 available'; \
fi; \

wget -O KEYS "$HTTPD_KEYS_URL"; \


#
# get distro - php
#
mkdir -p "$PHP_PREFIX"; \
cd "$PHP_PREFIX"; \

wget -O php-$PHP_VERSION.tar.xz "$PHP_URL"; \
wget -O php-$PHP_VERSION.tar.xz.asc "$PHP_ASC_URL"; \

# clean-up
apk del .wget-deps; \


#
# verify distro hashes - httpd
#
# see https://www.apache.org/dyn/closer.cgi#verify
# see https://www.apache.org/info/verification.html
if [ -f httpd-$HTTPD_VERSION.tar.bz2.md5 ]; then \
md5sum -c httpd-$HTTPD_VERSION.tar.bz2.md5; \
fi; \

if [ -f httpd-$HTTPD_VERSION.tar.bz2.sha1 ]; then \
sha1sum -c httpd-$HTTPD_VERSION.tar.bz2.sha1; \
fi; \

if [ -f httpd-$HTTPD_VERSION.tar.bz2.sha256 ]; then \
bytes=`du -k httpd-$HTTPD_VERSION.tar.bz2.sha256 | cut -f1`; \
if [ $bytes -ne 0 ]; then \
sha256sum -c httpd-$HTTPD_VERSION.tar.bz2.sha256; \
fi; \
fi; \


#
# verify distro hashes - php
#
if [ -n "$PHP_MD5" ]; then \
echo "$PHP_MD5 *php-$PHP_VERSION.tar.xz" | md5sum -c; \
fi; \

if [ -n "$PHP_SHA256" ]; then \
echo "$PHP_SHA256 *php-$PHP_VERSION.tar.xz" | sha256sum -c; \
fi; \


#
# verify distro signature - shared
#
apk add --no-cache --virtual .verify-deps \
gnupg \
; \

export GNUPGHOME="$(mktemp -d)"; \


#
# verify distro signature - httpd
# @todo: is this fingerprint verification necessary?
#
cd "$HTTPD_PREFIX"; \
gpg --import KEYS; \
gpg --verify httpd-$HTTPD_VERSION.tar.bz2.asc httpd-$HTTPD_VERSION.tar.bz2 2>&1 | tee distro-verify.txt; \
DISTRO_KEY=$(grep using distro-verify.txt | sed -e 's/gpg:\s*using ... key //'); \

DISTRO_FINGERPRINT=$(grep fingerprint distro-verify.txt | sed -e 's/Primary key fingerprint: //'); \
gpg --fingerprint $DISTRO_KEY 2>&1 | tee key-fingerprint.txt; \

if [ -z "$DISTRO_FINGERPRINT" ]; then \
echo 'no distro fingerprint'; \
exit 1; \
fi; \

if ! grep -Fq "$DISTRO_FINGERPRINT" key-fingerprint.txt; then \
echo 'distro and key fingerprints don’t match'; \
exit 1; \
fi; \

# cleanup
rm httpd-*.tar.bz2.*; \
rm KEYS; \
rm distro-verify.txt; \
rm key-fingerprint.txt; \


#
# verify distro signature - php
# @todo: is a fingerprint verification necessary?
#
cd "$PHP_PREFIX"; \
# pgp.mit.edu
# ha.pool.sks-keyservers.net
gpg --keyserver pgp.mit.edu --recv-keys $GPG_KEY; \
gpg --verify php-$PHP_VERSION.tar.xz.asc php-$PHP_VERSION.tar.xz 2>&1 | tee distro-verify.txt; \

DISTRO_FINGERPRINT=$(grep fingerprint distro-verify.txt | sed -e 's/Primary key fingerprint: //'); \
gpg --fingerprint $GPG_KEY 2>&1 | tee key-fingerprint.txt; \

if [ -z "$DISTRO_FINGERPRINT" ]; then \
echo 'no distro fingerprint'; \
exit 1; \
fi; \

if ! grep -Fq "$DISTRO_FINGERPRINT" key-fingerprint.txt; then \
echo 'distro and key fingerprints don’t match'; \
exit 1; \
fi; \

# clean-up
rm php-$PHP_VERSION.tar.xz.asc; \
rm distro-verify.txt; \
rm key-fingerprint.txt; \

# clean-up shared
rm -rf $GNUPGHOME; \
apk del .verify-deps; \


#
# extract distro - shared
#
apk add --no-cache --virtual .extract-deps \
tar \
xz \
; \


#
# extract distro - httpd
#
cd "$HTTPD_PREFIX"; \
mkdir src; \
tar -xf httpd-$HTTPD_VERSION.tar.bz2 -C src --strip-components=1; \

# clean-up
rm httpd-$HTTPD_VERSION.tar.bz2; \


#
# extract distro - php
#
cd "$PHP_PREFIX"; \
mkdir src; \
tar -xf php-$PHP_VERSION.tar.xz -C src --strip-components=1; \

# clean-up
rm php-$PHP_VERSION.tar.xz; \

# clean-up shared
apk del .extract-deps; \


#
# build - shared
#
apk add --no-cache --virtual .build-shared-deps \
coreutils \
dpkg \
dpkg-dev \
g++ \
make \
; \


#
# build - httpd
#
apk add --no-cache --virtual .build-httpd-deps \
apr-dev \
apr-util-dev \
; \

apk add --no-cache --virtual .configure-httpd-deps \
nghttp2-dev \
pcre-dev \
; \

cd "$HTTPD_PREFIX"; \
cd src; \
gnuArch="$(dpkg-architecture --query DEB_BUILD_GNU_TYPE)"; \

./configure \
--build="$gnuArch" \
--prefix="$HTTPD_PREFIX" \
; \

make -j "$(nproc)"; \
make install; \

# clean-up
cd ..; \
rm -rf src man manual; \


#
# build - php
#
apk add --no-cache --virtual .build-php-deps \
bison \
re2c \
; \

apk add --no-cache --virtual .configure-php-deps \
libedit-dev \
libxml2-dev \
; \

export CFLAGS="$PHP_CFLAGS" \
CPPFLAGS="$PHP_CPPFLAGS" \
LDFLAGS="$PHP_LDFLAGS"; \

cd "$PHP_PREFIX"; \
cd src; \
gnuArch="$(dpkg-architecture --query DEB_BUILD_GNU_TYPE)"; \

# http://php.net/manual/en/install.php
# http://php.net/manual/en/configure.about.php
./configure \
--build="$gnuArch" \
--with-config-file-path="$PHP_INI_DIR" \
--with-config-file-scan-dir="$PHP_INI_DIR/conf.d" \

--disable-cgi \

--with-apxs2 \
--with-libedit \
; \

make -j "$(nproc)"; \
make install; \
make clean; \

# https://github.com/docker-library/php/issues/443
pecl update-channels; \

# clean-up
cd ../../; \
rm -rf php; \
rm -rf /tmp/pear ~/.pearrc; \

# clean-up apk
# @todo: do .configure-httpd-deps and .configure-php-deps need to remain?
apk del .build-shared-deps; \
apk del .build-httpd-deps; \
apk del .build-php-deps; \


#
# add runtime dependencies - shared
#
runDeps="$( \
scanelf --needed --nobanner --recursive /usr/local \
| awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
| sort -u \
| xargs -r apk info --installed \
| sort -u \
)"; \

apk add --no-cache --virtual .httpd-php-rundeps $runDeps; \


#
# additional setup - httpd
#
sed -ri \
-e 's!^(\s*CustomLog)\s+\S+!\1 /proc/self/fd/1!g' \
-e 's!^(\s*ErrorLog)\s+\S+!\1 /proc/self/fd/2!g' \
"$HTTPD_PREFIX/conf/httpd.conf";


#
# finalise
#
EXPOSE 80
CMD ["httpd-foreground"]
