# 使用官方 PHP 8.5 FPM 版本作為基礎映像（Alpine 版本體積較小）
FROM php:8.5-fpm-alpine

# 設定映像維護者標籤
LABEL maintainer="dev-team"

# 設定時區環境變數為亞洲/台北
ENV TZ=Asia/Taipei

# 設定 PHP 相關環境變數，避免互動式安裝提示
ENV DEBIAN_FRONTEND=noninteractive

# 更新 Alpine 套件索引並安裝系統相依函式庫
RUN apk update && apk upgrade --no-cache \
    # 安裝時區資料套件，以支援 TZ 設定
    && apk add --no-cache tzdata \
    # 安裝 bash，取代預設的 sh，方便除錯
    && apk add --no-cache bash \
    # 安裝 curl，用於 HTTP 請求與健康檢查
    && apk add --no-cache curl \
    # 安裝 git，支援 Composer 透過 git 安裝套件
    && apk add --no-cache git \
    # 安裝 unzip，用於解壓縮 Composer 套件
    && apk add --no-cache unzip \
    # 安裝 zip，用於壓縮功能
    && apk add --no-cache zip \
    # 安裝 libpng 開發函式庫，為 GD 擴充套件提供 PNG 支援
    && apk add --no-cache libpng-dev \
    # 安裝 libjpeg-turbo 開發函式庫，為 GD 擴充套件提供 JPEG 支援
    && apk add --no-cache libjpeg-turbo-dev \
    # 安裝 freetype 開發函式庫，為 GD 提供字型渲染支援
    && apk add --no-cache freetype-dev \
    # 安裝 libzip 開發函式庫，為 zip 擴充套件提供支援
    && apk add --no-cache libzip-dev \
    # 安裝 icu 開發函式庫，為 intl 擴充套件提供國際化支援
    && apk add --no-cache icu-dev \
    # 安裝 oniguruma 開發函式庫，為 mbstring 擴充套件提供字元編碼支援
    && apk add --no-cache oniguruma-dev \
    # 安裝 PostgreSQL 客戶端開發函式庫，為 pdo_pgsql 擴充套件提供支援
    && apk add --no-cache libpq-dev \
    # 設定時區連結檔
    && cp /usr/share/zoneinfo/${TZ} /etc/localtime \
    # 將時區名稱寫入 /etc/timezone
    && echo "${TZ}" > /etc/timezone

# 設定並安裝 GD 擴充套件（支援 freetype 與 jpeg）
RUN docker-php-ext-configure gd \
    # 指定 freetype 函式庫目錄
    --with-freetype \
    # 指定 jpeg 函式庫目錄
    --with-jpeg

# 安裝 PHP 核心擴充套件
RUN docker-php-ext-install -j$(nproc) \
    # 安裝 PDO 核心擴充套件，提供資料庫抽象層
    pdo \
    # 安裝 PDO MySQL 驅動，支援 MySQL/MariaDB 連線
    pdo_mysql \
    # 安裝 PDO PostgreSQL 驅動，支援 PostgreSQL 連線
    pdo_pgsql \
    # 安裝 pgsql 擴充套件，提供原生 PostgreSQL 函數
    pgsql \
    # 安裝 mbstring 擴充套件，支援多位元組字串處理
    mbstring \
    # 安裝 GD 影像處理擴充套件
    gd \
    # 安裝 zip 擴充套件，支援 ZIP 壓縮/解壓縮
    zip \
    # 安裝 bcmath 擴充套件，支援高精度數學運算
    bcmath \
    # 安裝 intl 擴充套件，支援國際化與在地化功能
    intl \
    # 安裝 opcache 擴充套件，加速 PHP 執行效能
    opcache \
    # 安裝 pcntl 擴充套件，提供程序控制支援
    pcntl \
    # 安裝 posix 擴充套件，提供 POSIX 相容函數
    posix \
    # 安裝 sockets 擴充套件，支援底層 socket 通訊
    sockets \
    # 安裝 exif 擴充套件，支援讀取圖片 EXIF 資訊
    exif

# 透過 PECL 安裝 Redis 擴充套件，支援 Redis 快取
RUN pecl install redis \
    # 啟用 Redis 擴充套件
    && docker-php-ext-enable redis

# 透過 PECL 安裝 APCu 擴充套件，提供使用者層級快取
RUN pecl install apcu \
    # 啟用 APCu 擴充套件
    && docker-php-ext-enable apcu

# 複製自訂 PHP 設定檔至 php.d 目錄（覆蓋預設值）
COPY php.ini /usr/local/etc/php/conf.d/custom.ini

# 複製 php-fpm 進程池設定檔
COPY www.conf /usr/local/etc/php-fpm.d/www.conf

# 安裝 Composer（PHP 套件管理工具）
RUN curl -sS https://getcomposer.org/installer | php \
    # 將 Composer 移至全域執行路徑
    && mv composer.phar /usr/local/bin/composer \
    # 賦予 Composer 執行權限
    && chmod +x /usr/local/bin/composer

# 設定工作目錄為應用程式根目錄
WORKDIR /var/www/html

# 建立非 root 使用者 www，提升容器安全性
RUN addgroup -g 1000 www \
    # 建立 www 使用者並加入 www 群組
    && adduser -u 1000 -G www -s /bin/bash -D www

# 將工作目錄擁有權授予 www 使用者
RUN chown -R www:www /var/www/html

# 切換至非 root 使用者執行
USER www

# 暴露 PHP-FPM 預設監聽埠 9000
EXPOSE 9000

# 啟動 PHP-FPM 服務（前台模式）
CMD ["php-fpm"]
