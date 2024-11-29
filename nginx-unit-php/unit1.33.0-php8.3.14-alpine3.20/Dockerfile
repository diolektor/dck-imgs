# syntax=docker/dockerfile:1.4

ARG ALPINE_VERSION=3.20
ARG PHP_VERSION=8.3.14
ARG NJS_VERSION=0.8.4
ARG UNIT_VERSION=1.33.0

FROM php:${PHP_VERSION}-zts-alpine${ALPINE_VERSION} AS builder

ARG NJS_VERSION
ARG UNIT_VERSION

RUN set -eux \
    ; apk add  --no-cache curl unzip dpkg-dev build-base musl musl-utils dpkg git pcre2-dev openssl-dev \
    ; mkdir -p /usr/lib/unit/modules /usr/lib/unit/debug-modules \
    ; mkdir -p /usr/src/unit \
    ; git clone --depth 1 -b ${UNIT_VERSION}-1 https://github.com/nginx/unit \
    ; cd unit \
    ; NCPU="$(getconf _NPROCESSORS_ONLN)" \
    ; DEB_HOST_MULTIARCH="$(dpkg-architecture -q DEB_HOST_MULTIARCH)" \
    ; CC_OPT="$(DEB_BUILD_MAINT_OPTIONS="hardening=+all,-pie" DEB_CFLAGS_MAINT_APPEND="-Wp,-D_FORTIFY_SOURCE=2 -fPIC" dpkg-buildflags --get CFLAGS)" \
    ; LD_OPT="$(DEB_BUILD_MAINT_OPTIONS="hardening=+all,-pie" DEB_LDFLAGS_MAINT_APPEND="-Wl,--as-needed -pie" dpkg-buildflags --get LDFLAGS)" \
    ; CONFIGURE_ARGS_MODULES="--prefix=/usr \
                    --statedir=/var/lib/unit \
                    --control=unix:/var/run/control.unit.sock \
                    --runstatedir=/var/run \
                    --pid=/var/run/unit.pid \
                    --logdir=/var/log \
                    --log=/var/log/unit.log \
                    --tmpdir=/var/tmp \
                    --user=unit \
                    --group=unit \
                    --openssl \
                    --libdir=/usr/lib/$DEB_HOST_MULTIARCH" \
    ; CONFIGURE_ARGS="$CONFIGURE_ARGS_MODULES \
                    --njs" \
    ; make -j $NCPU -C pkg/contrib .njs \
    ; export PKG_CONFIG_PATH=$(pwd)/pkg/contrib/njs/build \
    ; ./configure $CONFIGURE_ARGS --cc-opt="$CC_OPT" --ld-opt="$LD_OPT" --modulesdir=/usr/lib/unit/debug-modules --debug \
    ; make -j $NCPU unitd \
    ; install -pm755 build/sbin/unitd /usr/sbin/unitd-debug \
    ; make clean \
    ; ./configure $CONFIGURE_ARGS --cc-opt="$CC_OPT" --ld-opt="$LD_OPT" --modulesdir=/usr/lib/unit/modules \
    ; make -j $NCPU unitd \
    ; install -pm755 build/sbin/unitd /usr/sbin/unitd \
    ; make clean \
    ; /bin/true \
    ; ./configure $CONFIGURE_ARGS_MODULES --cc-opt="$CC_OPT" --modulesdir=/usr/lib/unit/debug-modules --debug \
    ; ./configure php \
    ; make -j $NCPU php-install \
    ; make clean \
    ; ./configure $CONFIGURE_ARGS_MODULES --cc-opt="$CC_OPT" --modulesdir=/usr/lib/unit/modules \
    ; ./configure php \
    ; make -j $NCPU php-install \
    ; cd \
    ; rm -rf /usr/src/unit \
    ; mkdir -p /var/lib/unit/ \
    ; mkdir /docker-entrypoint.d/ \
    ; addgroup --system unit \
    ; adduser -S -D -H -G "unit" -g "unit user" -s /bin/false unit

COPY docker-entrypoint.sh /usr/local/bin/
COPY welcome.* /usr/share/unit/welcome/

STOPSIGNAL SIGTERM

ENTRYPOINT ["/usr/local/bin/docker-entrypoint.sh"]
EXPOSE 80
CMD ["unitd", "--no-daemon", "--control", "unix:/var/run/control.unit.sock"]


FROM php:${PHP_VERSION}-zts-alpine${ALPINE_VERSION} AS base

RUN apk update && apk upgrade --no-cache

RUN apk add --no-cache pcre2-dev openssl-dev musl-dev

COPY --link --from=builder /usr/sbin/unitd /usr/sbin/unitd
COPY --link --from=builder /usr/sbin/unitd-debug /usr/sbin/unitd-debug
COPY --link --from=builder /usr/lib/unit/ /usr/lib/unit/

COPY docker-entrypoint.sh /usr/local/bin/
COPY welcome.* /usr/share/unit/welcome/

RUN set -eux \
    ; addgroup --system unit \
    ; adduser -S -D -H -G "unit" -g "unit user" -s /bin/false unit

STOPSIGNAL SIGTERM

ENTRYPOINT ["/usr/local/bin/docker-entrypoint.sh"]
EXPOSE 80
CMD ["unitd", "--no-daemon", "--control", "unix:/var/run/control.unit.sock"]