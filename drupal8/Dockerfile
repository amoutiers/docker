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
# Change Apache DocumentRoot to suit drupal-composer/drupal-projet
# https://github.com/drupal-composer/drupal-project
ENV APACHE_DOCUMENT_ROOT /var/www/html/web
RUN sed -ri -e 's!/var/www/html!${APACHE_DOCUMENT_ROOT}!g' \
  /etc/apache2/sites-available/*.conf \
  && sed -ri -e 's!/var/www/!${APACHE_DOCUMENT_ROOT}!g' \
  /etc/apache2/apache2.conf /etc/apache2/conf-available/*.conf

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
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer
RUN echo "export PATH=$PATH:$HOME/.composer/vendor/bin" >> /root/.bashrc
RUN apt-get update \
  && apt-get install -y --no-install-recommends git


WORKDIR /var/www/html
