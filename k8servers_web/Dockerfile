FROM php:8.2-fpm-alpine

WORKDIR /var/www/html

RUN addgroup -g 101 -S nginx && \
    adduser -u 101 -S -G nginx -h /var/cache/nginx -s /sbin/nologin nginx

RUN apk add --no-cache --virtual .build-deps \
      git \
      $PHPIZE_DEPS \
      libzip-dev libpng-dev jpeg-dev freetype-dev libxml2-dev \
      linux-headers oniguruma-dev libssh2-dev openssl-dev \
  && docker-php-ext-configure gd --with-freetype --with-jpeg \
  && docker-php-ext-install -j"$(nproc)" \
       gd mysqli pdo_mysql zip soap sockets xml mbstring \
  && pecl channel-update pecl.php.net \
  && pecl install ssh2-1.3.1 \
  && docker-php-ext-enable ssh2 \
  && apk del .build-deps \
  && rm -rf /var/cache/apk/*

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

RUN mkdir -p /var/lib/php/sessions && \
    chown -R nginx:nginx /var/lib/php/sessions && \
    chmod -R 700 /var/lib/php/sessions

COPY --from=php:8.2-fpm-alpine /usr/local/etc/php/php.ini-production /usr/local/etc/php/php.ini
RUN sed -i 's#^;session.save_path = "/tmp"#session.save_path = "/var/lib/php/sessions"#' /usr/local/etc/php/php.ini && \
    sed -i 's#^session.save_path = "/tmp"#session.save_path = "/var/lib/php/sessions"#' /usr/local/etc/php/php.ini && \
    sed -i 's#^session.gc_probability = 0#session.gc_probability = 1#' /usr/local/etc/php/php.ini

RUN { \
      echo "session.cookie_domain = .k8servers.es"; \
      echo "session.cookie_secure = On"; \
      echo "session.cookie_httponly = On"; \
    } >> /usr/local/etc/php/php.ini

EXPOSE 9000
