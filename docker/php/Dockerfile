FROM php:8.5-fpm-alpine

RUN apk add --no-cache \
    nginx \
    supervisor \
    curl \
    libpng-dev \
    libzip-dev \
    zip \
    unzip \
    git \
    nodejs \
    npm

RUN curl -sSL https://github.com/mlocati/docker-php-extension-installer -o /usr/local/bin/install-php-extensions && \
    chmod +x /usr/local/bin/install-php-extensions && \
    install-php-extensions pdo_mysql bcmath gd zip redis

COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

WORKDIR /var/www/quiz

COPY package*.json composer*.json ./

RUN composer install --no-interaction --no-scripts --no-dev
RUN npm install

COPY . .

RUN npm run build

RUN composer install --no-interaction --optimize-autoloader --no-dev

COPY ./docker/nginx/app.conf /etc/nginx/http.d/default.conf
COPY ./docker/supervisord/supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY ./docker/php/zz-docker.conf /usr/local/etc/php-fpm.d/zz-docker.conf

RUN chown -R www-data:www-data /var/www/quiz/storage /var/www/quiz/bootstrap/cache

EXPOSE 80

CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]
