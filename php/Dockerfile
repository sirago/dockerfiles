FROM php:7.3.3-fpm-alpine3.9

ARG BUILD_CORES

ARG NGINX_VER=1.15.8

ARG NGINX_CONF=" \
    --prefix=/nginx \
    --sbin-path=/usr/local/sbin/nginx \
    --http-log-path=/nginx/logs/access.log \
    --error-log-path=/nginx/logs/error.log \
    --pid-path=/nginx/run/nginx.pid \
    --lock-path=/nginx/run/nginx.lock \
    --with-threads \
    --with-file-aio \
    --without-http_geo_module \
    --without-http_autoindex_module \
    --without-http_split_clients_module \
    --without-http_memcached_module \
    --without-http_empty_gif_module \
    --without-http_browser_module"

ARG CUSTOM_BUILD_PKGS=" \
    freetype-dev \
    openldap-dev \
    gmp-dev \
    icu-dev \
    postgresql-dev \
    libpng-dev \
    libwebp-dev \
    gd-dev \
    libjpeg-turbo-dev \
    libxpm-dev \
    libedit-dev \
    libxml2-dev \
    libbz2 \
    sqlite-dev \
    imap-dev"

ARG CUSTOM_PKGS=" \
    freetype \
    openldap \
    gmp \
    bzip2-dev \
    icu \
    libpq \
    c-client"

RUN BUILD_DEPS=" \
        linux-headers \
        libtool \
        build-base \
        pcre-dev \
        wget \
        gnupg \
        autoconf \
        gcc \
        g++ \
        libc-dev \
        make \
        pkgconf \
        curl-dev \
        ca-certificates \
        ${CUSTOM_BUILD_PKGS}" \
    set -ex; \
    apk --update upgrade --no-cache; \
    apk add --no-cache --virtual .build-deps ${BUILD_DEPS};

RUN NB_CORES=${BUILD_CORES-$(getconf _NPROCESSORS_CONF)} \
 && BUILD_DEPS=" \
    linux-headers \
    libtool \
    build-base \
    pcre-dev \
    wget \
    gnupg \
    autoconf \
    gcc \
    g++ \
    libc-dev \
    make \
    pkgconf \
    curl-dev \
    ca-certificates \
    libzip-dev \
    ${CUSTOM_BUILD_PKGS}" \
 && apk add --no-cache \
    s6 \
    su-exec \
    curl \
    libedit \
    libxml2 \
    libwebp \
    gd \
    pcre \
    ${CUSTOM_PKGS} \
 && wget http://nginx.org/download/nginx-${NGINX_VER}.tar.gz -O /tmp/nginx-${NGINX_VER}.tar.gz \
 && wget http://nginx.org/download/nginx-${NGINX_VER}.tar.gz.asc -O /tmp/nginx-${NGINX_VER}.tar.gz.asc \
 && mkdir -p /usr/src \
 && tar xzf /tmp/nginx-${NGINX_VER}.tar.gz -C /usr/src \
 && cd /usr/src/nginx-${NGINX_VER} \
 && ./configure --with-cc-opt="-O3 -fPIE -fstack-protector-strong" ${NGINX_CONF} \
 && make -j ${NB_CORES} \
 && make install \
 && ./configure --prefix=/usr/local \
 && make && make install && libtool --finish /usr/local/lib \
 && make -j ${NB_CORES} \
 && make install \
 && make clean

RUN apk add libzip-dev && docker-php-ext-install zip bcmath mysqli pdo pdo_mysql opcache gd

RUN apk del ${BUILD_DEPS} \
    && rm -rf /tmp/* /var/cache/apk/* /usr/src/*

RUN apk del .build-deps; \
    rm -rf /var/cache/apk/*; \
    rm -rf /app/config;

# Install Composer
RUN curl -sS https://getcomposer.org/installer | php \
    && mv composer.phar /usr/local/bin/ \
    && ln -s /usr/local/bin/composer.phar /usr/local/bin/composer

RUN composer global require hirak/prestissimo

COPY conf/services.d /etc/s6/

COPY conf/nginx/nginx.conf /usr/local/conf/nginx.conf
COPY conf/php/zz-docker.conf /usr/local/etc/php-fpm.d/zz-docker.conf

RUN chmod -R +x /etc/s6/* \
    && chmod +x /etc/s6/.s6-svscan/finish

RUN adduser -D nginx

RUN echo 'memory_limit = 512M' >> /usr/local/etc/php/conf.d/docker-php-memlimit.ini;

ENTRYPOINT ["/bin/s6-svscan", "/etc/s6"]
