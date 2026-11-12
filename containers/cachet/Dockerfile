FROM php:8.4-apache
LABEL description="Cachet Docker Image with MariaDB and PHP 8.4"
LABEL version="1.0"
LABEL org.opencontainers.image.description="Cachet Docker Image with MariaDB and PHP 8.4"

# Set DOCUMENT_ROOT to public directory from Laravel
ENV APACHE_DOCUMENT_ROOT=/var/www/html/public

# Copy custom apache and php configuration
COPY ./assets/apache/000-default.conf /etc/apache2/sites-available/000-default.conf
COPY ./assets/php/cachet.ini /usr/local/etc/php/conf.d/cachet.ini

# Install composer into container
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Update repo and packages and install apache & php
RUN apt-get update -y && apt-get upgrade -y
RUN a2enmod rewrite
RUN apt-get install tree nano libzip-dev libwebp-dev libfreetype6-dev libjpeg62-turbo-dev libpng-dev zlib1g-dev libicu-dev libpq-dev -y
RUN apt-get install npm -y

# Install PHP Modules
RUN docker-php-ext-install pdo_mysql \
  && docker-php-ext-install mysqli \
  && docker-php-ext-install zip \
  && docker-php-ext-install exif \
  && docker-php-ext-install gd \
  && docker-php-ext-install bcmath \
  && docker-php-ext-install intl \
  && docker-php-ext-install pcntl

# Set temporary to user root to copy files and set permissions
USER root

# Copy the application files to the Apache document root
RUN git clone -b 3.x https://github.com/cachethq/cachet.git /var/www/html && \
    rm -rf /var/www/html/.git && \
    rm -rf /var/www/html/.github && \
    rm -rf /var/www/html/.dockerignore && \
    rm -rf /var/www/html/.gitignore && \
    rm -rf /var/www/html/.gitattributes && \
    rm -rf /var/www/html/.editorconfig && \
    rm -rf /var/www/html/.travis.yml && \
    rm -rf /var/www/html/.styleci.yml && \
    rm -rf /var/www/html/.php_cs && \
    rm -rf /var/www/html/.php_cs.cache && \
    rm -rf /var/www/html/.scrutinizer.yml

# Set the correct permissions for the application files
RUN chown -R www-data:www-data /var/www

# Set Default User for Apache
USER www-data

# Use GitHub token for Composer because of rate-limitter
#RUN --mount=type=secret,id=github_token,env=GITHUB_TOKEN composer config -g github-oauth.github.com $GITHUB_TOKEN

# Install Composer dependencies and NPM packages
RUN composer install --no-dev -o

# RUN composer update cachethq/core
RUN composer update cachethq/core

# Publish the Cachet assets
RUN php artisan vendor:publish --tag=cachet

# Create symbolic link between storage and public directory
RUN php artisan storage:link