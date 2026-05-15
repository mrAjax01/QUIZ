FROM node:20-alpine AS frontend-builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .

RUN npm run build

FROM php:8.5-fpm-alpine

RUN apk add --no-cache \
    nginx \
    supervisor \
    curl \
    libpng-dev \
    libzip-dev \
    zip \
    unzip \
    git

RUN docker-php-ext-install pdo_mysql bcmath gd zip opcache

COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

WORKDIR /var/www/quiz

COPY . .

COPY --from=frontend-builder /app/public/build ./public/build

RUN composer install --no-interaction --optimize-autoloader --no-dev

COPY ./docker/nginx/app.conf /etc/nginx/http.d/default.conf
COPY ./docker/supervisord/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

RUN chown -R www-data:www-data /var/www/quiz/storage /var/www/quiz/bootstrap/cache

EXPOSE 80

CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]
