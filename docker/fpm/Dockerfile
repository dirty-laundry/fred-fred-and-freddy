FROM php:7.4-fpm-alpine AS base

# install extensions
RUN set -eux; \
  apk add --no-cache --upgrade bash curl; \
  apk add --no-cache --virtual .build-deps coreutils icu-dev zlib-dev libzip-dev oniguruma-dev; \
  docker-php-ext-configure zip; \
  docker-php-ext-configure intl; \
  docker-php-ext-install -j$(nproc) intl pcntl mbstring opcache zip; \
  runDeps="$( \
    scanelf --needed --nobanner --format '%n#p' --recursive /usr/local/lib/php/extensions \
      | tr ',' '\n' \
      | sort -u \
      | awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
    )"; \
   apk add --no-cache $runDeps; \
   apk del --no-network .build-deps

# remove default configuration files
RUN set -eux; \
  rm -f \
    /usr/local/etc/php-fpm.conf.default \
    /usr/local/etc/php-fpm.d/docker.conf \
    /usr/local/etc/php-fpm.d/www.conf \
    /usr/local/etc/php-fpm.d/www.conf.default \
    /usr/local/etc/php-fpm.d/zz-docker.conf

# copy configuration files from project
COPY /docker/fpm/php.ini /usr/local/etc/php/
COPY /docker/fpm/php-cli.ini /usr/local/etc/php/
COPY /docker/fpm/php-fpm.conf /usr/local/etc/
COPY /docker/fpm/php-fpm-pool.conf /usr/local/etc/php-fpm.d/

# set permissions on files/directories
RUN set -eux; \
  find /srv -type d -exec chmod 700 {} \+; \
  find /srv -type f -exec chmod 400 {} \+; \
  chown -R www-data:www-data /srv; \
  find /usr/local/etc -type d -exec chmod 755 {} \+; \
  find /usr/local/etc -type f -exec chmod 644 {} \+

WORKDIR /srv

EXPOSE 9000/tcp

STOPSIGNAL SIGQUIT

FROM base AS composer

RUN set -eux; \
  apk add --no-cache --upgrade openssh-client

COPY --from=composer:1 /usr/bin/composer /usr/bin/composer

FROM base AS production

COPY /docker/fpm/symfony.ini /usr/local/etc/php/conf.d/

COPY /backend/ /srv/
