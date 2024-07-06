FROM diolektor/php-cli-embed:php8.3-alpine3.20 AS builder

RUN apk add --no-cache curl unzip dpkg-dev build-base musl musl-utils dpkg

RUN set -eux \
  ; apk add --no-cache pcre2-dev openssl-dev

RUN set -eux \
    ; mkdir -p /usr/lib/unit/modules /usr/lib/unit/debug-modules \
    ; mkdir -p /usr/src/unit /usr/src/njs \
    ; cd /usr/src/njs \
    ; curl https://hg.nginx.org/njs/archive/0.8.2.tar.gz -o - | tar xvz \
    ; cd njs-0.8.2 \
    ; ./configure --no-libxml2 --no-zlib \
    ; make libnjs \
    ; export PKG_CONFIG_PATH=/usr/src/njs/njs-0.8.2/build \
    ; cd /usr/src/unit \
    ; curl -O https://unit.nginx.org/download/unit-1.32.1.tar.gz \
    ; curl -O https://unit.nginx.org/download/unit-1.32.1.tar.gz.sha512 \
    ; sha512sum -c unit-1.32.1.tar.gz.sha512 \
    ; tar -xvzf unit-1.32.1.tar.gz \
    ; cd unit-1.32.1 \
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


FROM diolektor/php-cli-embed:php8.3-alpine3.20 AS base

RUN apk update && apk upgrade --no-cache

RUN apk add --no-cache pcre2-dev openssl-dev musl-dev

COPY --from=builder /usr/sbin/unitd /usr/sbin/unitd
COPY --from=builder /usr/sbin/unitd-debug /usr/sbin/unitd-debug
COPY --from=builder /usr/lib/unit/ /usr/lib/unit/

COPY docker-entrypoint.sh /usr/local/bin/
COPY welcome.* /usr/share/unit/welcome/

RUN set -eux \
    ; addgroup --system unit \
    ; adduser -S -D -H -G "unit" -g "unit user" -s /bin/false unit

STOPSIGNAL SIGTERM

ENTRYPOINT ["/usr/local/bin/docker-entrypoint.sh"]
EXPOSE 80
CMD ["unitd", "--no-daemon", "--control", "unix:/var/run/control.unit.sock"]