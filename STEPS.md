create a dockerfile

# Use PHP 8.3 (Laravel 12 requires PHP 8.2+)
FROM php:8.3-fpm

# Configure apt to handle proxy/network issues
RUN echo 'Acquire::http::Pipeline-Depth "0";' > /etc/apt/apt.conf.d/99custom && \
    echo 'Acquire::http::No-Cache=True;' >> /etc/apt/apt.conf.d/99custom && \
    echo 'Acquire::BrokenProxy=true;' >> /etc/apt/apt.conf.d/99custom && \
    echo 'Acquire::https::Verify-Peer "false";' >> /etc/apt/apt.conf.d/99custom && \
    echo 'Acquire::https::Verify-Host "false";' >> /etc/apt/apt.conf.d/99custom

# Try to use alternative mirrors and HTTPS
RUN rm -f /etc/apt/sources.list.d/debian.sources && \
    echo "deb https://deb.debian.org/debian trixie main" > /etc/apt/sources.list && \
    echo "deb https://deb.debian.org/debian trixie-updates main" >> /etc/apt/sources.list && \
    echo "deb https://security.debian.org/debian-security trixie-security main" >> /etc/apt/sources.list

# Install system dependencies
RUN apt-get update && apt-get install -y \
    git curl libpng-dev libonig-dev libxml2-dev zip unzip \
    && docker-php-ext-install pdo_mysql mbstring exif pcntl bcmath gd

# Install Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Set working directory
WORKDIR /var/www

# Copy the Laravel files
COPY . .

# Install Laravel dependencies
RUN composer install --no-interaction --prefer-dist --optimize-autoloader

# Set folder permissions
RUN chown -R www-data:www-data /var/www/storage /var/www/bootstrap/cache

EXPOSE 9000
CMD ["php-fpm"]



create a docker-compose.yml

run 
- docker compose up -d build
- docker compose exec app sh
- php artisan key:generate
- php artisan migrate 

go to localhost:8080