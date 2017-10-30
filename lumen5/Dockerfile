ARG  COMPOSER_VERSION=latest
ARG  PHP_VERSION

# Build composer
FROM composer:${COMPOSER_VERSION} as build_composer

# Build lumen
FROM php:${PHP_VERSION}-apache

# Update package list
RUN apt-get update

# Setup Apache
ENV APACHE_DOCUMENT_ROOT /var/www/html/public
RUN sed -ri -e 's!/var/www/html!${APACHE_DOCUMENT_ROOT}!g' \
  /etc/apache2/sites-available/*.conf
RUN sed -ri -e 's!/var/www/!${APACHE_DOCUMENT_ROOT}!g' \
  /etc/apache2/apache2.conf /etc/apache2/conf-available/*.conf

# Setup PHP
RUN docker-php-ext-install \
  mbstring \
  pdo \
  pdo_mysql

# Install composer from composer build
COPY --from=build_composer /usr/bin/composer /usr/bin/composer

WORKDIR /var/www/html
