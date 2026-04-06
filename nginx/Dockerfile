# 使用官方 Nginx Alpine 版本作為基礎映像（體積小、效能好）
FROM nginx:alpine

# 設定映像維護者標籤
LABEL maintainer="dev-team"

# 設定時區環境變數為亞洲/台北
ENV TZ=Asia/Taipei

# 安裝時區資料套件，並設定系統時區
RUN apk add --no-cache tzdata \
    # 建立時區連結檔
    && cp /usr/share/zoneinfo/${TZ} /etc/localtime \
    # 將時區名稱寫入 /etc/timezone
    && echo "${TZ}" > /etc/timezone \
    # 移除時區資料套件以縮減映像大小
    && apk del tzdata

# 移除預設的 Nginx 設定檔（改用自訂設定）
RUN rm /etc/nginx/conf.d/default.conf

# 複製自訂 Nginx 主設定檔（覆蓋預設值）
COPY nginx.conf /etc/nginx/nginx.conf

# 複製站台設定檔至 conf.d 目錄（Nginx 會自動載入此目錄下所有 .conf 檔）
COPY conf.d/default.conf /etc/nginx/conf.d/default.conf

# 建立 Nginx 日誌目錄，確保日誌可正常寫入
RUN mkdir -p /var/log/nginx

# 對外暴露 HTTP 埠號 80
EXPOSE 80

# 對外暴露 HTTPS 埠號 443（若需要 SSL 可開啟）
EXPOSE 443

# 啟動 Nginx（daemon off 使其在前台執行，保持容器存活）
CMD ["nginx", "-g", "daemon off;"]
