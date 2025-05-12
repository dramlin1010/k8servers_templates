FROM php:8.2-fpm-alpine

ENV APP_ENV=production \
    NGINX_WORKER_PROCESSES=auto \
    NGINX_WORKER_CONNECTIONS=1024

RUN apk add --no-cache \
      nginx \
      supervisor \
      curl \
      git \
      libzip-dev \
      libpng-dev \
      jpeg-dev \
      freetype-dev \
      libxml2-dev \
      linux-headers \
      $PHPIZE_DEPS oniguruma-dev \
  && docker-php-ext-configure gd --with-freetype --with-jpeg \
  && docker-php-ext-install -j$(nproc) \
      gd \
      mysqli \
      pdo_mysql \
      zip \
      soap \
      sockets \
      xml \
      mbstring \
  && pecl install ssh2-1.3.1 \
  && docker-php-ext-enable ssh2 \
  && rm -rf /var/cache/apk/*

RUN sed -i \
      -e 's|error_log .*|error_log /dev/stderr;|' \
      -e 's|access_log .*|access_log /dev/stdout;|' \
      /etc/nginx/nginx.conf \
  && sed -i \
      -e 's|error_log = .*|error_log = /dev/stderr|' \
      /usr/local/etc/php-fpm.conf \
  && sed -i \
      -e 's|access.log = .*|access.log = /dev/stdout|' \
      /usr/local/etc/php-fpm.d/www.conf

COPY ./nginx/default.conf /etc/nginx/conf.d/default.conf

RUN mkdir -p /run/php /var/log/nginx /var/lib/nginx /var/www/html \
  && chown -R nginx:nginx /run/php /var/log/nginx /var/lib/nginx /var/www/html

COPY ./supervisor/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

WORKDIR /var/www/html
EXPOSE 80
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]
