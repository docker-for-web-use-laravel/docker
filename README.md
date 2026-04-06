# Docker Laravel 環境

基於 Docker Compose 的 Laravel 應用程式開發與部署環境，包含 Nginx、PHP-FPM、PHP Worker 與 PostgreSQL 四個服務。

## 架構說明

```
瀏覽器
  │  HTTP :80
  ▼
Nginx — 接收 Web 請求，靜態資源直接回應，PHP 請求轉發至 php-fpm
  │
  ▼
PHP-FPM — 執行 Laravel 應用程式，處理動態請求
  │
  ▼
PostgreSQL — 資料庫

PHP Worker — 獨立運行，由 Supervisor 管理 Laravel Queue 與 Schedule 任務
```

| 服務 | 說明 | 對外埠號 |
|------|------|----------|
| nginx | Web 伺服器，反向代理至 php-fpm | 80, 443 |
| php-fpm | PHP 8.5 FastCGI 處理器 | — |
| php-worker | PHP 8.5 背景任務執行器（Supervisor） | — |
| db | PostgreSQL 資料庫 | 5432 |

## 目錄結構

```
docker/
├── docker-compose.yml
├── app/                  # Laravel 專案放置於此
├── nginx/
│   ├── Dockerfile
│   ├── nginx.conf
│   └── conf.d/
│       └── default.conf
├── php-fpm/
│   ├── Dockerfile
│   ├── php.ini
│   └── www.conf
├── php-worker/
│   ├── Dockerfile
│   ├── php.ini
│   └── supervisord.conf
└── postgres_data/        # 資料庫資料（自動產生）
```

## 使用方式

### 1. 放入 Laravel 專案

將 Laravel 專案放至 `app/` 目錄，或複製現有專案：

```bash
cp -r /your/laravel/project ./app
```

### 2. 設定環境變數

複製並修改 Laravel 的 `.env`：

```bash
cp app/.env.example app/.env
```

資料庫連線設定對應如下：

```env
DB_CONNECTION=pgsql
DB_HOST=db
DB_PORT=5432
DB_DATABASE=d_postgres
DB_USERNAME=user
DB_PASSWORD=password
```

### 3. 啟動服務

```bash
docker compose up -d
```

### 4. 初始化 Laravel

```bash
# 安裝 Composer 套件
docker exec php-fpm composer install

# 產生應用程式金鑰
docker exec php-fpm php artisan key:generate

# 執行資料庫遷移
docker exec php-fpm php artisan migrate
```

### 5. 開啟瀏覽器

前往 [http://localhost](http://localhost)

## 常用指令

```bash
# 啟動所有服務
docker compose up -d

# 停止所有服務
docker compose down

# 查看服務狀態
docker compose ps

# 查看服務日誌
docker compose logs -f nginx
docker compose logs -f php-fpm
docker compose logs -f php-worker

# 進入容器
docker exec -it php-fpm bash
docker exec -it php-worker bash

# 連線至資料庫
docker exec -it postgres_db psql -U user -d d_postgres

# 重新建置映像（修改 Dockerfile 後執行）
docker compose build
docker compose up -d
```

## Worker 設定

背景任務由 `php-worker` 容器內的 Supervisor 管理，設定檔位於 `php-worker/supervisord.conf`。

預設已包含兩個範例程序，請依專案調整 `command=` 的路徑與參數：

| 程序名稱 | 說明 |
|----------|------|
| queue-worker | Laravel Queue Worker（`artisan queue:work`） |
| schedule-runner | Laravel Scheduler（`artisan schedule:run`） |

查看 Worker 執行狀態：

```bash
docker exec -it php-worker supervisorctl status
```
