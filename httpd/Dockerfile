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
# COPY
#
COPY httpd-foreground /usr/local/bin/


#
# RUN
# one run to create one layer
#
RUN \
set -xe; \
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
# get distro - httpd
#
apk add --no-cache --virtual .wget-deps \
ca-certificates \
wget \
; \

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
# verify distro signature - httpd
# @todo: is this fingerprint verification necessary?
#
apk add --no-cache --virtual .verify-deps \
gnupg \
; \

export GNUPGHOME="$(mktemp -d)"; \
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

# clean-up
rm -rf $GNUPGHOME; \
rm httpd-*.tar.bz2.*; \
rm KEYS; \
rm distro-verify.txt; \
rm key-fingerprint.txt; \
apk del .verify-deps; \


#
# extract distro - httpd
#
apk add --no-cache --virtual .extract-deps \
tar \
; \

cd "$HTTPD_PREFIX"; \
mkdir src; \
tar -xf httpd-$HTTPD_VERSION.tar.bz2 -C src --strip-components=1; \

# clean-up
rm httpd-$HTTPD_VERSION.tar.bz2; \
apk del .extract-deps; \


#
# build - httpd
#
apk add --no-cache --virtual .build-shared-deps \
coreutils \
dpkg \
dpkg-dev \
g++ \
make \
; \

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
rm -r src man manual; \

# clean-up apk
# @todo: do .configure-httpd-deps need to remain?
apk del .build-shared-deps; \
apk del .build-httpd-deps; \


#
# add runtime dependencies - httpd
#
runDeps="$( \
scanelf --needed --nobanner --recursive /usr/local \
| awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
| sort -u \
| xargs -r apk info --installed \
| sort -u \
)"; \

apk add --no-cache --virtual .httpd-rundeps $runDeps; \


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
