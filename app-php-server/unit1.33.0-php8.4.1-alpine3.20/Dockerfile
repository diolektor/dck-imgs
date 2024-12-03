ARG COMPOSER_VERSION=latest

FROM composer:${COMPOSER_VERSION} AS composer


# BASE IMAGE
FROM diolektor/nginx-unit-php:unit1.33.0-php8.4.1-alpine3.20 AS base

RUN apk add --update linux-headers && apk update && apk upgrade --no-cache

ARG TIMEZONE="UTC"

RUN apk add --no-cache curl unzip file

ADD --chmod=0755 https://github.com/mlocati/docker-php-extension-installer/releases/latest/download/install-php-extensions /usr/local/bin/

# install php extentions
RUN install-php-extensions opcache exif gd intl pdo_mysql pdo_pgsql zip redis pcntl

RUN cp /usr/local/etc/php/php.ini-production /usr/local/etc/php/php.ini

# set timezone
RUN set -eux \
    ln -snf /usr/share/zoneinfo/${TIMEZONE} /etc/localtime \
    ; echo ${TIMEZONE} > /etc/timezone \
    ; printf '[PHP]\ndate.timezone = "%s"\n', "$TIMEZONE" > $PHP_INI_DIR/conf.d/tzone.ini \
    ; "date"

# hide X-Powered-By
RUN echo "expose_php=off" > $PHP_INI_DIR/conf.d/expose.ini

# configure server
COPY config.json /docker-entrypoint.d/config.json

# setup composer
COPY --link --from=composer /usr/bin/composer /usr/bin/composer
ENV COMPOSER_ALLOW_SUPERUSER=1
RUN set -eux \
    ; composer clear-cache
ENV PATH="${PATH}:/root/.composer/vendor/bin"

RUN mkdir -p /var/lib/unit

# root dir
RUN mkdir -p /var/www/html

WORKDIR /var/www/html

RUN set -eux \
    ; mkdir -p /var/www/html \
    ; echo "<?php phpinfo();" > /var/www/html/index.php

EXPOSE 80

STOPSIGNAL SIGTERM
ENTRYPOINT ["/usr/local/bin/docker-entrypoint.sh"]
CMD ["unitd", "--no-daemon", "--control", "unix:/var/run/control.unit.sock"]



FROM base AS dev

RUN cp /usr/local/etc/php/php.ini-development /usr/local/etc/php/php.ini

COPY php-ext-xdebug.ini $PHP_INI_DIR/conf.d/php-ext-xdebug.ini

RUN install-php-extensions xdebug
