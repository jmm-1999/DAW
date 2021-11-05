FROM php:8.0-fpm

# Update OS and install common dev tools
RUN apt-get update
RUN apt-get install -y nano

# Install PHP extensions needed
RUN docker-php-ext-install -j$(nproc) mysqli pdo_mysql

# Instala Xdebug
RUN pecl install xdebug \
    && docker-php-ext-enable xdebug

# Instala composer
COPY --from=composer /usr/bin/composer /usr/bin/composer

# Set working directory to web files
WORKDIR /var/www/html