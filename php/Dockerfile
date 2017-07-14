FROM alpine:3.6

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
COPY docker-php-entrypoint /usr/local/bin/


#
# RUN
# one run to create one layer
#
RUN \
set -xe; \
mkdir -p $PHP_INI_DIR/conf.d; \
chmod +x /usr/local/bin/docker-php-entrypoint; \


#
# get distro - php
#
apk add --no-cache --virtual .wget-deps \
ca-certificates \
wget \
; \

mkdir -p "$PHP_PREFIX"; \
cd "$PHP_PREFIX"; \

wget -O php-$PHP_VERSION.tar.xz "$PHP_URL"; \
wget -O php-$PHP_VERSION.tar.xz.asc "$PHP_ASC_URL"; \

# clean-up
apk del .wget-deps; \


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
# verify distro signature - php
# @todo: is a fingerprint verification necessary?
#
apk add --no-cache --virtual .verify-deps \
gnupg \
; \

export GNUPGHOME="$(mktemp -d)"; \
cd "$PHP_PREFIX"; \

# pgp.mit.edu
# ha.pool.sks-keyservers.net
gpg --keyserver ha.pool.sks-keyservers.net --recv-keys $GPG_KEY; \
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
rm -rf $GNUPGHOME; \
rm php-$PHP_VERSION.tar.xz.asc; \
rm distro-verify.txt; \
rm key-fingerprint.txt; \
apk del .verify-deps; \


#
# extract distro - php
#
apk add --no-cache --virtual .extract-deps \
tar \
xz \
; \

cd "$PHP_PREFIX"; \
mkdir src; \
tar -xf php-$PHP_VERSION.tar.xz -C src --strip-components=1; \

# clean-up
rm php-$PHP_VERSION.tar.xz; \
apk del .extract-deps; \


#
# build - php
#
apk add --no-cache --virtual .build-shared-deps \
coreutils \
dpkg \
dpkg-dev \
g++ \
make \
; \

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

--with-libedit \
; \

make -j "$(nproc)"; \
make install; \
make clean; \

# @todo: does this need to be sorted out
#{ find /usr/local/bin /usr/local/sbin -type f -perm +0111 -exec strip --strip-all '{}' + || true; }; \

# https://github.com/docker-library/php/issues/443
pecl update-channels; \

# clean-up
cd ../../; \
rm -rf php; \
rm -rf /tmp/pear ~/.pearrc; \

# clean-up apk
# @todo: do .configure-php-deps need to remain?
apk del .build-shared-deps; \
apk del .build-php-deps; \


#
# add runtime dependencies - php
#
runDeps="$( \
scanelf --needed --nobanner --recursive /usr/local \
| awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
| sort -u \
| xargs -r apk info --installed \
| sort -u \
)"; \

apk add --no-cache --virtual .php-rundeps $runDeps;


#
# finalise
#
ENTRYPOINT ["docker-php-entrypoint"]
CMD ["php", "-a"]
