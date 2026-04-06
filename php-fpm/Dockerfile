# 使用 PHP 8.4 穩定版本
FROM php:8.4-fpm-alpine

# 基本環境設定
ENV TZ=Asia/Taipei
WORKDIR /var/www/html

# 1. 安裝 Laravel 運行絕對必要的系統套件
# 整合 apk update 邏輯，並加入編譯 Redis 所需的臨時工具 ($PHPIZE_DEPS)
RUN apk add --no-cache \
    git \
    unzip \
    icu-dev \
    libpq-dev \
    tzdata \
    linux-headers

# 2. 安裝核心套件
RUN docker-php-ext-install pdo_pgsql pgsql intl bcmath opcache sockets \
    && docker-php-source delete

# 3. 使用 .build-deps 建立臨時編譯環境，安裝完後立刻刪除以縮小體積
RUN apk add --no-cache --virtual .build-deps $PHPIZE_DEPS \
    && pecl install redis \
    && docker-php-ext-enable redis \
    && apk del .build-deps

# 4. 安裝 Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# 5. 權限與使用者
RUN addgroup -g 1000 www && adduser -u 1000 -G www -D www \
    && chown -R www:www /var/www/html

# 6. 複製設定檔
COPY php.ini /usr/local/etc/php/conf.d/custom.ini
COPY www.conf /usr/local/etc/php-fpm.d/www.conf

USER www
EXPOSE 9000
CMD ["php-fpm"]