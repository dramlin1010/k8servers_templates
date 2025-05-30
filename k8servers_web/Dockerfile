FROM php:8.2-fpm-alpine

WORKDIR /var/www/html

RUN addgroup -g 101 -S nginx && \
    adduser -u 101 -S -G nginx -h /var/cache/nginx -s /sbin/nologin nginx

RUN apk add --no-cache \
      git \
      libzip-dev libpng-dev jpeg-dev freetype-dev libxml2-dev \
      linux-headers oniguruma-dev libssh2-dev openssl-dev \
  && docker-php-ext-configure gd --with-freetype --with-jpeg \
  && docker-php-ext-install -j"$(nproc)" \
       gd mysqli pdo_mysql zip soap sockets xml mbstring \
  && pecl install ssh2-1.3.1 \
  && docker-php-ext-enable ssh2

RUN apk add --no-cache --virtual .phpize-deps-mongodb $PHPIZE_DEPS \
  && pecl channel-update pecl.php.net \
  && pecl install mongodb \
  && docker-php-ext-enable mongodb \
  && apk del .phpize-deps-mongodb

RUN rm -rf /var/cache/apk/*

RUN sed -i -E \
      -e 's|^listen\s*=\s*.*|listen = 0.0.0.0:9000|' \
      -e 's|^(\s*);?listen.allowed_clients\s*=\s*.*|;\1listen.allowed_clients = 127.0.0.1|' \
      -e 's|^(\s*)user\s*=\s*.*|\1user = nginx|' \
      -e 's|^(\s*)group\s*=\s*.*|\1group = nginx|' \
      -e 's|^(\s*)listen.owner\s*=\s*.*|\1listen.owner = nginx|' \
      -e 's|^(\s*)listen.group\s*=\s*.*|\1listen.group = nginx|' \
      -e 's|^(\s*)catch_workers_output\s*=\s*yes|\1catch_workers_output = yes|' \
      -e 's|^(\s*);?php_admin_value\[error_log\]\s*=\s*.*|php_admin_value[error_log] = /dev/stderr|' \
      -e '/^\s*access.log\s*=/d' \
      /usr/local/etc/php-fpm.d/www.conf && \
    sed -i -E \
      -e 's|^(\s*);?error_log\s*=\s*.*|\1error_log = /dev/stderr|' \
      -e 's|^(\s*)daemonize\s*=\s*yes|\1daemonize = no|' \
      /usr/local/etc/php-fpm.conf

RUN mkdir -p /run/php && \
    chown -R nginx:nginx /run/php

EXPOSE 9000
