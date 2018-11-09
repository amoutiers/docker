ARG COMPOSER_VERSION=latest
ARG PHP_VERSION=7.1


###
# Build composer
###
FROM composer:${COMPOSER_VERSION} as build_composer


###
# Build backdrop container
###
FROM php:${PHP_VERSION}-apache

# Setup apache
RUN a2enmod rewrite

# install the PHP extensions we need
RUN set -ex; \
  \
  if command -v a2enmod; then \
    a2enmod rewrite; \
  fi; \
  \
  savedAptMark="$(apt-mark showmanual)"; \
  \
  apt-get update; \
  apt-get install -y --no-install-recommends \
    libjpeg-dev \
    libpng-dev \
    libpq-dev \
  ; \
  \
  docker-php-ext-configure gd --with-png-dir=/usr --with-jpeg-dir=/usr; \
  docker-php-ext-install -j "$(nproc)" \
    gd \
    opcache \
    pdo_mysql \
    pdo_pgsql \
    zip \
  ; \
  \
# reset apt-mark's "manual" list so that "purge --auto-remove" will remove all build dependencies
  apt-mark auto '.*' > /dev/null; \
  apt-mark manual $savedAptMark; \
  ldd "$(php -r 'echo ini_get("extension_dir");')"/*.so \
    | awk '/=>/ { print $3 }' \
    | sort -u \
    | xargs -r dpkg-query -S \
    | cut -d: -f1 \
    | sort -u \
    | xargs -rt apt-mark manual; \
  \
  apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
  rm -rf /var/lib/apt/lists/*

# set recommended PHP.ini settings
# see https://secure.php.net/manual/en/opcache.installation.php
RUN { \
    echo 'opcache.memory_consumption=128'; \
    echo 'opcache.interned_strings_buffer=8'; \
    echo 'opcache.max_accelerated_files=4000'; \
    echo 'opcache.revalidate_freq=60'; \
    echo 'opcache.fast_shutdown=1'; \
    echo 'opcache.enable_cli=1'; \
} > /usr/local/etc/php/conf.d/opcache-recommended.ini

# Install MariaDB client
RUN apt-get update \
  && apt-get install -y mariadb-client

# Installation ssmtp
RUN apt-get update \
  && apt-get install -y ssmtp mailutils

# Install composer from lastest image
COPY --from=build_composer /usr/bin/composer /usr/bin/composer
RUN echo "export PATH=$PATH:$HOME/.composer/vendor/bin" >> /root/.bashrc
RUN apt-get update \
  && apt-get install -y --no-install-recommends git

# Install drush
ARG DRUSH_VERSION=8.x
RUN composer global require "drush/drush:${DRUSH_VERSION}" --prefer-dist

# Install backdrop command for drush
RUN git clone \
  https://github.com/backdrop-contrib/drush.git \
  /root/.composer/vendor/drush/drush/commands/backdrop


WORKDIR /var/www/html
