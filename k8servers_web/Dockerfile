FROM php:8.2-fpm-alpine

ENV APP_ENV=production \
    NGINX_WORKER_PROCESSES=auto \
    NGINX_WORKER_CONNECTIONS=1024

RUN apk add --no-cache \
      nginx supervisor curl git \
      libzip-dev libpng-dev jpeg-dev freetype-dev libxml2-dev \
      linux-headers oniguruma-dev libssh2-dev openssl-dev \
      $PHPIZE_DEPS \
  && docker-php-ext-configure gd --with-freetype --with-jpeg \
  && docker-php-ext-install -j"$(nproc)" \
       gd mysqli pdo_mysql zip soap sockets xml mbstring \
  && pecl install ssh2-1.3.1 \
  && docker-php-ext-enable ssh2 \
  && rm -rf /var/cache/apk/*

RUN echo "\
user nginx; \
worker_processes auto; \
error_log /dev/stderr warn; \
pid /run/nginx.pid; \
\
events { \
    worker_connections 1024; \
} \
\
http { \
    include       /etc/nginx/mime.types; \
    default_type  application/octet-stream; \
    log_format  main  '\$remote_addr - \$remote_user [\$time_local] \"\$request\" ' \
                      '\$status \$body_bytes_sent \"\$http_referer\" ' \
                      '\"\$http_user_agent\" \"\$http_x_forwarded_for\"'; \
    access_log  /dev/stdout main; \
    sendfile        on; \
    keepalive_timeout  65; \
    include /etc/nginx/conf.d/*.conf; \
} \
" > /etc/nginx/nginx.conf

RUN sed -i -E \
      -e 's|^(\s*);?error_log\s*=\s*.*|\1error_log = /dev/stderr|' \
      -e 's|^(\s*)daemonize\s*=\s*yes|\1daemonize = no|' \
      -e 's|^(\s*);?pid\s*=\s*.*|\1pid = /run/php/php-fpm.pid|' \
      /usr/local/etc/php-fpm.conf && \
    sed -i -E \
      -e 's|^(\s*)user\s*=\s*.*|\1user = nginx|' \
      -e 's|^(\s*)group\s*=\s*.*|\1group = nginx|' \
      -e 's|^(\s*)listen\s*=\s*.*|\1listen = /run/php/php-fpm.sock|' \
      -e 's|^(\s*)listen.owner\s*=\s*.*|\1listen.owner = nginx|' \
      -e 's|^(\s*)listen.group\s*=\s*.*|\1listen.group = nginx|' \
      -e 's|^(\s*)listen.mode\s*=\s*.*|\1listen.mode = 0660|' \
      -e 's|^(\s*)catch_workers_output\s*=\s*yes|\1catch_workers_output = yes|' \
      -e '/^\s*php_admin_value\[error_log\]\s*=/d' \
      -e '/^\s*access.log\s*=/d' \
      /usr/local/etc/php-fpm.d/www.conf

COPY ./nginx/default.conf /etc/nginx/conf.d/default.conf
RUN sed -i -E \
      -e '/^\s*access_log\s+.*/d' \
      -e '/^\s*error_log\s+.*/d' \
      /etc/nginx/conf.d/default.conf

RUN mkdir -p /run/php /var/www/html /var/lib/nginx/tmp \
    && chown -R nginx:nginx /run/php /var/www/html /var/lib/nginx \
    && chmod 755 /var/www/html

COPY ./supervisor/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

WORKDIR /var/www/html
EXPOSE 80
CMD ["/usr/bin/supervisord","-c","/etc/supervisor/conf.d/supervisord.conf"]
