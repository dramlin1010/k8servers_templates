FROM php:8.2-fpm-alpine

ENV APP_ENV=production
ENV NGINX_WORKER_PROCESSES=auto
ENV NGINX_WORKER_CONNECTIONS=1024

RUN apk update && apk add --no-cache \
    nginx supervisor curl git \
    libzip-dev libpng-dev jpeg-dev freetype-dev libxml2-dev libssh2-dev linux-headers \
    && rm -rf /var/cache/apk/*

RUN apk add --no-cache --virtual .build-deps $PHPIZE_DEPS oniguruma-dev \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j$(nproc) \
    gd mysqli pdo_mysql zip soap sockets xml mbstring && pecl install ssh2-1.3.1 && docker-php-ext-enable ssh2 && apk del .build-deps

COPY ./nginx/default.conf /etc/nginx/http.d/default.conf
RUN rm -f /etc/nginx/nginx.conf && \
    mkdir -p /var/log/nginx /var/lib/nginx && \
    chown -R nginx:nginx /var/log/nginx /var/lib/nginx

COPY ./supervisor/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

RUN mkdir -p /run/php && \
    chown -R nginx:nginx /run/php

RUN mkdir -p /var/www/html && chown nginx:nginx /var/www/html

WORKDIR /var/www/html

EXPOSE 80
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]
