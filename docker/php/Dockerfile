FROM php:7.2-fpm-alpine

RUN echo http://dl-cdn.alpinelinux.org/alpine/edge/community/ >> /etc/apk/repositories
RUN apk update --no-cache \
    && apk add --no-cache build-base \
    git \
    shadow \
    autoconf \
    libmcrypt-dev \
    postgresql-dev \
    freetype-dev \
    imagemagick-dev \
    libjpeg-turbo-dev \
    libpng-dev \
    libtool \
    make \
    g++ \
    libc-dev \
    pcre-dev \
    mysql-client \
    openjdk7-jre

RUN docker-php-ext-install pdo_mysql pdo_pgsql mysqli opcache \
    && docker-php-ext-install -j"$(getconf _NPROCESSORS_ONLN)" iconv \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j"$(getconf _NPROCESSORS_ONLN)" gd \
    && pecl install imagick \
    && docker-php-ext-enable imagick

RUN { \
    echo 'opcache.memory_consumption=128'; \
    echo 'opcache.interned_strings_buffer=8'; \
    echo 'opcache.max_accelerated_files=4000'; \
    echo 'opcache.revalidate_freq=5'; \
    echo 'opcache.fast_shutdown=1'; \
    echo 'opcache.enable_cli=1'; \
    } > /usr/local/etc/php/conf.d/opcache-recommended.ini

RUN echo "expose_php=0" > /usr/local/etc/php/php.ini

  RUN git clone https://github.com/php-memcached-dev/php-memcached /usr/src/php/ext/memcached \
    && cd /usr/src/php/ext/memcached && git checkout -b php7 origin/php7 \
    && apk add \
    cyrus-sasl-dev \
    libmemcached-dev \
    zlib-dev \
    && docker-php-ext-install memcached \
    && php -m | grep memcached


ARG INSTALL_XDEBUG=false
RUN if [ ${INSTALL_XDEBUG} = true ]; then \
    pecl install xdebug-2.6.1 && \
    docker-php-ext-enable xdebug \
    ;fi


# Configure version constraints
ENV PHP_ENABLE_XDEBUG=0 \
    VERSION_COMPOSER_ASSET_PLUGIN=^1.4.2 \
    VERSION_PRESTISSIMO_PLUGIN=^0.3.7 \
    PATH=/app:/app/vendor/bin:/root/.composer/vendor/bin:$PATH \
    TERM=linux \
    COMPOSER_ALLOW_SUPERUSER=1

RUN usermod -u 1000 www-data
COPY ./www.conf /usr/local/etc/php-fpm.d/www.conf
COPY ./xdebug.ini /usr/local/etc/php/conf.d/xdebug.ini
COPY ./custom.ini /usr/local/etc/php/conf.d/custom.ini
RUN apk del build-base autoconf g++ libtool make
RUN rm -rf /tmp/* /var/cache/apk/*

# Install composer
RUN curl -sS https://getcomposer.org/installer | \
  php -- --install-dir=/usr/local/bin --filename=composer

ARG GITHUB_API_TOKEN
USER www-data
RUN composer config -g github-oauth.github.com ${GITHUB_API_TOKEN}
RUN composer global require --optimize-autoloader \
  "fxp/composer-asset-plugin:${VERSION_COMPOSER_ASSET_PLUGIN}" \
  "hirak/prestissimo:${VERSION_PRESTISSIMO_PLUGIN}" && \
  composer global dumpautoload --optimize && \
  composer clear-cache
