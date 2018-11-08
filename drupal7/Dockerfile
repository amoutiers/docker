ARG COMPOSER_VERSION=latest
ARG PHP_VERSION


###
# Build composer
###
FROM composer:${COMPOSER_VERSION} as build_composer


###
# Build drupal container
###
FROM php:${PHP_VERSION}-apache

# Setup apache
RUN a2enmod rewrite

# install the PHP extensions we need
RUN set -ex \
  && buildDeps=' \
    libjpeg62-turbo-dev \
    libpng12-dev \
    libpq-dev \
  ' \
  && apt-get update && apt-get install -y --no-install-recommends $buildDeps && rm -rf /var/lib/apt/lists/* \
  && docker-php-ext-configure gd \
    --with-jpeg-dir=/usr \
    --with-png-dir=/usr \
  && docker-php-ext-install -j "$(nproc)" gd mbstring opcache pdo pdo_mysql pdo_pgsql zip \
# PHP Warning:  PHP Startup: Unable to load dynamic library '/usr/local/lib/php/extensions/no-debug-non-zts-20151012/gd.so' - libjpeg.so.62: cannot open shared object file: No such file or directory in Unknown on line 0
# PHP Warning:  PHP Startup: Unable to load dynamic library '/usr/local/lib/php/extensions/no-debug-non-zts-20151012/pdo_pgsql.so' - libpq.so.5: cannot open shared object file: No such file or directory in Unknown on line 0
  && apt-mark manual \
    libjpeg62-turbo \
    libpq5 \
  && apt-get purge -y --auto-remove $buildDeps

# Install MariaDB client
RUN apt-get update \
    && apt-get install -y mariadb-client

# Installation ssmtp
RUN apt-get install -y ssmtp mailutils

# Install composer from lastest image
COPY --from=build_composer /usr/bin/composer /usr/bin/composer
RUN echo "export PATH=$PATH:$HOME/.composer/vendor/bin" >> /root/.bashrc
RUN apt-get update \
  && apt-get install -y --no-install-recommends git

# Install drush
ARG DRUSH_VERSION=8.x
RUN composer global require "drush/drush:${DRUSH_VERSION}" --prefer-dist


WORKDIR /var/www/html