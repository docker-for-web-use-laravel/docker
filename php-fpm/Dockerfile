# 使用 PHP 8.4 穩定版本，避免 8.5 實驗版的編譯 Bug
FROM php:8.4-fpm-alpine

# 基本環境設定
ENV TZ=Asia/Taipei
WORKDIR /var/www/html

# 1. 僅安裝 Laravel 運行絕對必要的系統套件
# 包含：git/unzip (Composer 用), libpq (PostgreSQL 用), icu-dev (intl 用)
RUN apk add --no-cache \
    git unzip icu-dev libpq-dev tzdata linux-headers \
    && cp /usr/share/zoneinfo/${TZ} /etc/localtime \
    && echo "${TZ}" > /etc/timezone

# 2. 安裝 Laravel 最低要求的 PHP 擴充套件
# 移除所有非必要的編譯過程，僅保留核心連線與邏輯運算套件
RUN docker-php-ext-install pdo_pgsql pgsql intl bcmath opcache sockets \
    && docker-php-source delete

# 3. 透過 PECL 安裝 Redis (Laravel 預設快取/隊列強烈建議使用)
RUN pecl install redis && docker-php-ext-enable redis

# 4. 安裝 Composer (從官方 Image 直接複製，最快且不會出錯)
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# 5. 安全性與目錄權限
# 建立 www 使用者並確保 UID 與 WSL2 一致
RUN addgroup -g 1000 www && adduser -u 1000 -G www -D www \
    && chown -R www:www /var/www/html

# 6. 複製設定檔 (確保這兩個檔案存在於您的目錄中)
COPY php.ini /usr/local/etc/php/conf.d/custom.ini
COPY www.conf /usr/local/etc/php-fpm.d/www.conf

USER www
EXPOSE 9000
CMD ["php-fpm"]
