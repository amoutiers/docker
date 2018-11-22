ARG COMPOSER_VERSION=latest
ARG PHP_VERSION=7.2


###
# Build composer
###
FROM composer:${COMPOSER_VERSION} as build_composer


###
# Build drupal container
###
FROM php:${PHP_VERSION}-apache-stretch

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
  echo 'opcache.enable=1'; \
} > /usr/local/etc/php/conf.d/opcache-recommended.ini

ARG BUILD_DEV=0
RUN \
  if [ "x$BUILD_DEV" = "x1" ]; then \
    ln -s /usr/local/etc/php/php.ini-development /usr/local/etc/php/php.ini; \
    echo 'opcache.enable_cli=0' >> /usr/local/etc/php/conf.d/opcache-recommended.ini; \
    echo 'opcache.enable=0' >> /usr/local/etc/php/conf.d/opcache-recommended.ini; \
    curl -o /tmp/xhprof.tgz https://codeload.github.com/tideways/php-xhprof-extension/tar.gz/v4.1.6 \
      && tar xzf /tmp/xhprof.tgz -C /tmp \
      && cd /tmp/php-xhprof-extension-4.1.6 \
      && phpize \
      && ./configure \
      && make \
      && make install \
      && echo 'extension=tideways.so' > /usr/local/etc/php/conf.d/xhprof.ini; \
    pecl install xdebug-2.6.0 \
      && docker-php-ext-enable xdebug; \
  else \
    ln -s /usr/local/etc/php/php.ini-production /usr/local/etc/php/php.ini; \
  fi

# Install MariaDB client
RUN apt-get update \
  && apt-get install -y --no-install-recommends mariadb-client

# Configure mail
RUN apt-get update \
   && apt-get install -y --no-install-recommends ssmtp mailutils

# Install composer from lastest image
COPY --from=build_composer /usr/bin/composer /usr/bin/composer
RUN echo "export PATH=$PATH:$HOME/.composer/vendor/bin" >> /root/.bashrc
RUN apt-get update \
  && apt-get install -y --no-install-recommends git

# Install drush
ARG DRUSH_VERSION=8.x
RUN composer global require "drush/drush:${DRUSH_VERSION}" --prefer-dist


EXPOSE 80 9000

WORKDIR /var/www/html
