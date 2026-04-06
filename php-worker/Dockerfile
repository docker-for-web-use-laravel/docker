# 使用 PHP 8.4 CLI 穩定版本，Worker 不需 FPM
FROM php:8.4-cli-alpine

# 基本環境設定
ENV TZ=Asia/Taipei
WORKDIR /var/www/html

# 1. 安裝 Worker 運行絕對必要的系統套件
# 包含：git/unzip (Composer 用), libpq (PostgreSQL 用), 
# icu-dev (intl 用), supervisor (管理背景進程)
RUN apk add --no-cache \
    git unzip icu-dev libpq-dev tzdata supervisor linux-headers \
    && cp /usr/share/zoneinfo/${TZ} /etc/localtime \
    && echo "${TZ}" > /etc/timezone

# 2. 安裝 Laravel 最低要求的 PHP 擴充套件
# 包含：pcntl/posix (Worker 與 Supervisor 訊號處理必要)
RUN docker-php-ext-install pdo_pgsql pgsql intl bcmath opcache sockets pcntl posix \
    && docker-php-source delete

# 3. 透過 PECL 安裝 Redis (Laravel 佇列通常依賴 Redis)
RUN pecl install redis && docker-php-ext-enable redis

# 4. 安裝 Composer (官方映像檔直接複製)
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# 5. 安全性設定與日誌目錄
RUN addgroup -g 1000 www && adduser -u 1000 -G www -D www \
    && mkdir -p /var/log/supervisor \
    && chown -R www:www /var/log/supervisor /var/www/html

# 6. 複製設定檔 (請確保這些檔案存在於專案目錄中)
COPY php.ini /usr/local/etc/php/conf.d/custom.ini
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf

# Supervisor 建議以 root 啟動以管理進程
USER root

# 啟動 Supervisor
CMD ["/usr/bin/supervisord", "-n", "-c", "/etc/supervisor/conf.d/supervisord.conf"]
