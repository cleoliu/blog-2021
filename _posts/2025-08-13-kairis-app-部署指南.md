---
layout: post
title: "Kairis App - 部署指南"
date: 2025-08-13 12:45:56 +0800
categories: [General]
tags: [general]
description: "Kairis App - 部署指南"
image: https://res.cloudinary.com/dsvl326mi/image/upload/v1755060356/blog_covers/icon-192x192_c7vr3t.png---

# Web-Based AI Service 部署指南

本指南將提供一個詳細的步驟，用於部署一個基於 Python (Django) 的 Web AI 服務到 Linux 伺服器環境。該服務將利用 Gunicorn 作為應用伺服器，Nginx 作為反向代理，並搭配 PostgreSQL 數據庫。

## 1. 部署概述

本部署指南的目標是將一個基於 Django 框架的 AI 應用程式，從開發環境部署到生產環境，使其能夠穩定、高效地對外提供服務。

### 1.1 部署目標

*   將 AI 應用程式代碼部署到生產伺服器。
*   配置 Gunicorn 以運行 Django 應用程式。
*   配置 Nginx 作為反向代理，處理 HTTP 請求並提供靜態文件。
*   配置 PostgreSQL 數據庫以存儲應用程式數據。
*   確保應用程式在生產環境中的穩定性和安全性。
*   啟用 SSL/TLS 加密以保護數據傳輸。

### 1.2 系統架構

該 AI 服務的典型生產部署架構如下：

*   **作業系統 (OS)**：Ubuntu Server LTS 或其他基於 Debian/Red Hat 的 Linux 發行版。
*   **應用程式框架**：Python 3.x, Django。
*   **AI 模型**: 預訓練的 AI 模型（例如，基於 TensorFlow/PyTorch 的模型）由 Django 應用程式載入並提供推斷服務。
*   **數據庫**：PostgreSQL (用於 Django ORM、用戶數據等)。
*   **應用伺服器**：Gunicorn (Python WSGI HTTP Server)，負責運行 Django 應用程式。
*   **網頁伺服器/反向代理**：Nginx (處理客戶端請求，將動態請求轉發給 Gunicorn，並直接提供靜態文件)。
*   **安全**：防火牆 (UFW)、SSL/TLS (Certbot/Let's Encrypt)。

```text
[客戶端] <--- HTTPS/HTTP ---> [Nginx] <--- HTTP (Unix Socket) ---> [Gunicorn] <---> [Django AI App]
                                 ^                                      ^
                                 |                                      |
                                 +---------- 提供靜態文件 -----------+
                                                                        |
                                                                        +---> [PostgreSQL 數據庫]
                                                                        +---> [AI 模型文件]
```text
## 2. 環境準備

在開始部署之前，請確保您的伺服器滿足以下環境要求並已準備好必要的工具。

### 2.1 系統需求

*   **操作系統**: 推薦使用 Ubuntu Server 20.04 LTS 或更新版本。
*   **CPU**: 建議多核心處理器（例如，2 核或更多，取決於 AI 模型複雜度和負載）。
*   **記憶體 (RAM)**: 建議 8GB 或更多，尤其是在載入大型 AI 模型時。
*   **硬碟空間**: 至少 50GB，用於操作系統、應用程式代碼、AI 模型文件、數據庫和日誌文件。
*   **網路**: 穩定的網路連接，確保端口 22 (SSH)、80 (HTTP) 和 443 (HTTPS) 可用。

### 2.2 依賴項目

在您的伺服器上安裝以下軟體包：

*   **Python 3.8+**: 應用程式運行環境。
*   **`pip`**: Python 包管理工具。
*   **`venv` (或 `virtualenv`)**: 用於創建隔離的 Python 環境。
*   **`nginx`**: 反向代理伺服器。
*   **`git`**: 用於從版本控制系統克隆應用程式代碼。
*   **`postgresql-client`**: 用於連接到 PostgreSQL 數據庫 (如果數據庫在同一伺服器，可能需要 `postgresql-server`)。
*   **`build-essential`**: 編譯一些 Python 包（例如 `psycopg2-binary`）可能需要。
*   **`libpq-dev`**: PostgreSQL 開發庫，用於編譯 Python PostgreSQL 驅動。

### 2.3 配置要求

*   **SSH 訪問**: 擁有伺服器的 SSH 訪問權限，並使用非 `root` 用戶，該用戶具備 `sudo` 權限。
*   **域名**: 一個已註冊的域名，並配置 DNS A 記錄指向您的伺服器 IP 地址。
*   **SSL 憑證**: 建議使用 Let's Encrypt 頒發的免費 SSL/TLS 憑證。
*   **防火牆**: 配置防火牆規則，只開放必要的端口。

## 3. 部署步驟

請按照以下詳細步驟在您的伺服器上部署 AI 服務。

### 3.1 步驟 1 - SSH 連接與系統更新

首先，通過 SSH 連接到您的伺服器，並更新系統。

```bash
ssh your_user@your_server_ip
sudo apt update && sudo apt upgrade -y
```text
### 3.2 步驟 2 - 安裝核心依賴

安裝應用程式運行所需的基礎軟體包。

```bash
sudo apt install -y python3-pip python3-venv nginx git postgresql-client build-essential libpq-dev
```text
### 3.3 步驟 3 - 克隆應用程式代碼

將您的 AI 應用程式代碼從 Git 倉庫克隆到伺服器。建議克隆到 `/opt` 目錄下。

```bash
cd /opt
sudo git clone https://github.com/your-org/your-ai-app.git
sudo chown -R your_user:your_user your-ai-app # 將目錄權限賦予您的用戶
cd your-ai-app
```text
### 3.4 步驟 4 - 創建並激活虛擬環境

為應用程式創建一個獨立的 Python 虛擬環境，以避免包衝突。

```bash
python3 -m venv venv
source venv/bin/activate
```text
### 3.5 步驟 5 - 安裝 Python 包依賴

激活虛擬環境後，使用 `pip` 安裝 `requirements.txt` 中列出的所有 Python 包，包括 Django、Gunicorn 和 AI 相關庫。

```bash
pip install --upgrade pip
pip install -r requirements.txt
```text
**`requirements.txt` 示例:**

```text
Django
djangorestframework
gunicorn
psycopg2-binary
tensorflow # 或者 pytorch, 根據您的AI模型框架選擇
transformers # 如果您使用基於 transformer 的模型
scikit-learn # 如果您使用 scikit-learn 模型
# 其他您的應用所需的包
```text
### 3.6 步驟 6 - 配置 Django 應用程式與數據庫

#### 3.6.1 創建數據庫和用戶

如果您在同一台伺服器上運行 PostgreSQL，請創建一個新的數據庫和用戶供您的 Django 應用程式使用。

```bash
sudo -u postgres psql
# 在 psql 提示符下執行以下命令：
CREATE DATABASE your_ai_app_db;
CREATE USER your_db_user WITH PASSWORD 'your_db_password';
ALTER ROLE your_db_user SET client_encoding TO 'utf8';
ALTER ROLE your_db_user SET default_transaction_isolation TO 'read committed';
ALTER ROLE your_db_user SET timezone TO 'UTC';
GRANT ALL PRIVILEGES ON DATABASE your_ai_app_db TO your_db_user;
\q
```text
#### 3.6.2 配置 Django `settings.py`

編輯您的 Django 專案的 `settings.py` 文件，進行生產環境配置。

```python
# /opt/your-ai-app/your_ai_app/settings.py

import os
from pathlib import Path

# 將 DEBUG 設置為 False，生產環境絕不能為 True
DEBUG = False

# 設置允許訪問的域名或 IP 地址
ALLOWED_HOSTS = ['your_domain.com', 'www.your_domain.com', 'your_server_ip']

# 數據庫配置
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.environ.get('DB_NAME', 'your_ai_app_db'),
        'USER': os.environ.get('DB_USER', 'your_db_user'),
        'PASSWORD': os.environ.get('DB_PASSWORD', 'your_db_password'),
        'HOST': os.environ.get('DB_HOST', 'localhost'), # 如果數據庫在其他伺服器，請修改為其IP或域名
        'PORT': os.environ.get('DB_PORT', '5432'),
    }
}

# 靜態文件設置
STATIC_URL = '/static/'
STATIC_ROOT = os.path.join(BASE_DIR, 'static_collected') # 靜態文件收集到此目錄

# AI 模型路徑配置 (示例)
# 假設您的AI模型文件在項目根目錄下的 ai_models 目錄中
AI_MODEL_PATH = os.path.join(BASE_DIR, 'ai_models', 'your_model.h5') # 或者 .pt, .pb 等

# 其他生產環境相關配置，例如 SECRET_KEY
SECRET_KEY = os.environ.get('DJANGO_SECRET_KEY', 'a-very-secret-key-for-production-only-please-change-this')

# 日誌配置 (可選，但強烈建議)
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'file': {
            'level': 'INFO',
            'class': 'logging.handlers.RotatingFileHandler',
            'filename': os.path.join(BASE_DIR, 'logs', 'django.log'),
            'maxBytes': 1024 * 1024 * 5,  # 5 MB
            'backupCount': 5,
            'formatter': 'verbose',
        },
    },
    'formatters': {
        'verbose': {
            'format': '{levelname} {asctime} {module} {process:d} {thread:d} {message}',
            'style': '{',
        },
    },
    'loggers': {
        'django': {
            'handlers': ['file'],
            'level': 'INFO',
            'propagate': True,
        },
        'your_ai_app': { # 替換為您的應用程式名稱
            'handlers': ['file'],
            'level': 'INFO',
            'propagate': False,
        },
    },
}
```text
#### 3.6.3 執行數據庫遷移和收集靜態文件

```bash
# 確保您仍在虛擬環境中 (source venv/bin/activate)
python manage.py makemigrations
python manage.py migrate
python manage.py collectstatic --noinput # 收集所有靜態文件到 STATIC_ROOT
```text
### 3.7 步驟 7 - 配置 Gunicorn 服務

創建一個 Systemd 服務文件，以便 Gunicorn 能夠作為後台服務運行並在伺服器啟動時自動啟動。

```bash
sudo nano /etc/systemd/system/your_ai_app.service
```text
將以下內容粘貼到文件中，並替換 `your_user`、`your-ai-app` 和 `your_ai_app` (WSGI 文件名) 為您的實際值。

```ini
[Unit]
Description=Gunicorn instance to serve your_ai_app
After=network.target

[Service]
User=your_user # 替換為您的非root用戶
Group=www-data # 或者您的用戶組，如 your_user
WorkingDirectory=/opt/your-ai-app
ExecStart=/opt/your-ai-app/venv/bin/gunicorn --workers 3 --bind unix:/run/your_ai_app.sock your_ai_app.wsgi:application
# --workers - Gunicorn 進程數，建議為 (2 * CPU核心數) + 1
# --bind - 綁定到 Unix Socket，這是 Nginx 和 Gunicorn 通信的最佳方式
# your_ai_app.wsgi -application - 替換為您的 Django 專案的 WSGI 模塊路徑

Restart=on-failure # 當服務失敗時自動重啟
StandardOutput=append:/opt/your-ai-app/logs/gunicorn_access.log # 可選：Gunicorn 訪問日誌
StandardError=append:/opt/your-ai-app/logs/gunicorn_error.log # 可選：Gunicorn 錯誤日誌

[Install]
WantedBy=multi-user.target
```text
保存並退出 (`Ctrl+X`, `Y`, `Enter`)。然後啟用並啟動 Gunicorn 服務：

```bash
sudo systemctl daemon-reload
sudo systemctl start your_ai_app
sudo systemctl enable your_ai_app
sudo systemctl status your_ai_app
```text
檢查 Gunicorn 狀態，確保它正在運行。如果失敗，請查看日誌 (`sudo journalctl -u your_ai_app`).

### 3.8 步驟 8 - 配置 Nginx 反向代理

創建一個 Nginx 虛擬主機配置文件，將所有請求轉發到 Gunicorn 服務。

```bash
sudo nano /etc/nginx/sites-available/your_ai_app
```text
將以下內容粘貼到文件中，並替換 `your_domain.com` 和 `your_ai_app`。

```nginx
server {
    listen 80;
    server_name your_domain.com www.your_domain.com; # 替換為您的域名

    # 處理 favicon.ico 請求
    location = /favicon.ico { access_log off; log_not_found off; }

    # 靜態文件服務
    location /static/ {
        root /opt/your-ai-app; # 指向您Django專案的根目錄
        expires 30d; # 緩存靜態文件
        add_header Cache-Control "public";
    }

    # 動態請求轉發到 Gunicorn
    location / {
        include proxy_params;
        proxy_pass http://unix:/run/your_ai_app.sock; # Gunicorn 綁定的 Unix Socket
    }

    # 可選：日誌配置
    access_log /var/log/nginx/your_ai_app_access.log;
    error_log /var/log/nginx/your_ai_app_error.log;
}
```text
保存並退出。然後創建一個符號鏈接，激活這個配置，並測試 Nginx 配置。

```bash
sudo ln -s /etc/nginx/sites-available/your_ai_app /etc/nginx/sites-enabled/
sudo nginx -t # 測試 Nginx 配置語法
sudo systemctl restart nginx
```text
### 3.9 步驟 9 - 配置防火牆 (UFW)

如果您的伺服器上啟用了防火牆 (如 UFW)，請確保開放 SSH、HTTP 和 HTTPS 端口。

```bash
sudo ufw allow 'Nginx Full' # 允許 HTTP 和 HTTPS
# 如果只想允許 HTTP, 可以用 sudo ufw allow Nginx HTTP
sudo ufw allow 'OpenSSH'
sudo ufw enable
sudo ufw status # 確認防火牆規則
```text
### 3.10 步驟 10 - (可選) 配置 SSL/TLS (使用 Certbot)

強烈建議為您的網站配置 SSL/TLS 以實現安全連接。Certbot 是一個簡單易用的工具，可以自動為 Nginx 配置 Let's Encrypt 證書。

```bash
# 安裝 snapd (如果尚未安裝)
sudo apt update
sudo apt install snapd

# 確保 snapd 是最新版本
sudo snap install core
sudo snap refresh core

# 安裝 Certbot
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot

# 運行 Certbot 並為 Nginx 配置證書
sudo certbot --nginx -d your_domain.com -d www.your_domain.com

# 測試證書自動續訂
sudo certbot renew --dry-run
```text
Certbot 會自動修改 Nginx 配置，添加 HTTPS 支持，並設置自動續訂。完成後，您的網站將可通過 `https://your_domain.com` 訪問。

## 4. 驗證測試

部署完成後，請執行以下步驟以驗證應用程式是否正常運行。

### 4.1 服務狀態檢查

檢查 Gunicorn 和 Nginx 服務是否都在運行。

```bash
sudo systemctl status your_ai_app
sudo systemctl status nginx
```text
### 4.2 應用程式日誌檢查

查看應用程式、Gunicorn 和 Nginx 的日誌，尋找任何錯誤或異常。

```bash
# Gunicorn 和 Django 應用程式日誌
sudo journalctl -u your_ai_app -f # 實時查看 Gunicorn 系統日誌
# 或者查看您在 Gunicorn .service 文件中配置的日誌文件
# sudo tail -f /opt/your-ai-app/logs/gunicorn_access.log
# sudo tail -f /opt/your-ai-app/logs/gunicorn_error.log
# sudo tail -f /opt/your-ai-app/logs/django.log # 如果您在settings.py中配置了Django文件日誌

# Nginx 日誌
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log
```text
### 4.3 網站訪問測試

在瀏覽器中訪問您的域名 (例如 `https://your_domain.com`)，確認網站能正常載入並顯示內容。

### 4.4 API 端點測試

如果您的 AI 服務提供 API 端點，可以使用 `curl` 或 Postman 等工具進行測試。

```bash
# 假設您有一個 /api/predict/ 端點接收 JSON 數據
curl -X POST -H "Content-Type: application/json" \
     -d '{"text": "你好，AI！"}' \
     https://your_domain.com/api/predict/
```text
### 4.5 數據庫連接測試

如果您的應用程式需要與數據庫交互，請通過註冊用戶、發布內容等方式，驗證數據庫讀寫功能是否正常。

## 5. 常見問題排解

在部署過程中，您可能會遇到以下一些常見問題及其解決方案。

### 5.1 Gunicorn 無法啟動或報錯

*   **問題**: `sudo systemctl status your_ai_app` 顯示服務啟動失敗。
*   **解決方案**:
    *   **查看日誌**: 最重要的一步是查看 Gunicorn 的系統日誌 (`sudo journalctl -u your_ai_app -f`) 或您在服務文件中配置的日誌文件。
    *   **檢查路徑**: 確保 `WorkingDirectory` 和 `ExecStart` 中的所有路徑都是正確且可訪問的。
    *   **權限問題**: 確保 `User` 帳戶對 `/opt/your-ai-app` 目錄及其內容有讀寫權限。
    *   **依賴缺失**: 再次檢查 `requirements.txt` 中所有依賴是否都已成功安裝。特別是像 `psycopg2-binary` 這樣需要編譯的庫，確保 `build-essential` 和 `libpq-dev` 已安裝。
    *   **虛擬環境**: 確認 `ExecStart` 中的 Python 和 Gunicorn 路徑指向虛擬環境內部的可執行文件 (例如 `/opt/your-ai-app/venv/bin/gunicorn`)。
    *   **WSGI 模塊**: 確保 `your_ai_app.wsgi:application` 中的 `your_ai_app` 是您 Django 專案的正確名稱。

### 5.2 Nginx 502 Bad Gateway

*   **問題**: 瀏覽器顯示 502 Bad Gateway 錯誤。
*   **解決方案**:
    *   **Gunicorn 運行狀態**: `502` 錯誤通常表示 Nginx 無法連接到 Gunicorn。首先確認 Gunicorn 服務正在運行 (`sudo systemctl status your_ai_app`)。
    *   **Socket 路徑**: 檢查 Nginx 配置 (`/etc/nginx/sites-available/your_ai_app`) 中的 `proxy_pass http://unix:/run/your_ai_app.sock;` 路徑是否與 Gunicorn 服務文件中的 `--bind unix:/run/your_ai_app.sock` 路徑完全一致。
    *   **Socket 權限**: 確認 `/run/your_ai_app.sock` 文件存在，且 Nginx 用戶 (通常是 `www-data`) 有權限訪問它。可以嘗試將 Gunicorn 服務的 `Group` 設置為 `www-data`。
    *   **Nginx 錯誤日誌**: 查看 Nginx 的錯誤日誌 (`sudo tail -f /var/log/nginx/error.log`)，它會提供連接失敗的詳細原因。

### 5.3 靜態文件 (CSS/JS/圖片) 無法載入

*   **問題**: 網站載入但沒有樣式，圖片顯示錯誤。
*   **解決方案**:
    *   **`collectstatic`**: 確保您已運行 `python manage.py collectstatic --noinput`，這會將所有靜態文件收集到 `STATIC_ROOT` 指定的目錄。
    *   **Nginx `location /static/` 配置**: 檢查 Nginx 配置中 `location /static/` 的 `root` 路徑是否正確指向 `STATIC_ROOT` 指定的目錄 (`/opt/your-ai-app/static_collected` 或您實際設置的目錄)。
    *   **Django `settings.py`**: 確認 `STATIC_URL` 和 `STATIC_ROOT` 在 `settings.py` 中配置正確。

### 5.4 數據庫連接問題

*   **問題**: 應用程式無法連接到數據庫，或數據無法存儲。
*   **解決方案**:
    *   **數據庫服務狀態**: 確認 PostgreSQL 服務正在運行 (`sudo systemctl status postgresql`)。
    *   **`settings.py` 配置**: 檢查 `settings.py` 中 `DATABASES` 配置的 `NAME`, `USER`, `PASSWORD`, `HOST`, `PORT` 是否正確。
    *   **用戶權限**: 確保數據庫用戶 `your_db_user` 對 `your_ai_app_db` 數據庫有正確的權限。
    *   **防火牆**: 如果數據庫在不同的伺服器上，請檢查數據庫伺服器的防火牆是否允許來自應用程式伺服器的連接（PostgreSQL 默認端口 5432）。

### 5.5 AI 模型載入失敗或推斷錯誤

*   **問題**: 應用程式啟動時報錯或 AI 功能無法正常工作。
*   **解決方案**:
    *   **模型路徑**: 檢查 `settings.py` 中 `AI_MODEL_PATH` 配置是否正確，並確認模型文件實際存在於指定路徑。
    *   **環境**: 確保生產環境中安裝的 AI 相關庫 (TensorFlow, PyTorch, Transformers 等) 版本與開發環境兼容，且模型文件與這些庫的版本兼容。
    *   **資源**: 載入大型 AI 模型可能需要大量記憶體。檢查伺服器記憶體使用情況 (`free -h` 或 `htop`)，如果記憶體不足可能導致模型載入失敗。
    *   **應用程式日誌**: 仔細檢查應用程式日誌 (Django 日誌或 Gunicorn 日誌)，查找與模型載入或推斷相關的詳細錯誤信息。

### 5.6 Django 模板語法錯誤 (僅在開發或特定情境)

*   **問題**: 在渲染 Django 模板時遇到語法錯誤或解析問題。
*   **解決方案**:
    *   確保您的模板語法遵循 Django 規範。
    *   **變數**: 使用 `{{{{ variable_name }}}}` 顯示變數。
    *   **標籤**: 使用 `{{% tag_name %}}` 執行邏輯（例如循環、條件、URL生成等）。
    *   **示例 (用於 Django HTML 模板文件)**:
        ```html
        {{% raw %}}
        <!DOCTYPE html>
        <html lang="zh-Hant">
        <head>
            <meta charset="UTF-8">
            <title>{{{{ title }}}}</title>
            {{% load static %}}
            <link rel="stylesheet" href="{{{{ static 'css/style.css' }}}}">
        </head>
        <body>
            <h1>歡迎, {{{{ user.username }}}}!</h1>

            {{% if messages %}}
                <ul class="messages">
                    {{% for message in messages %}}
                        <li class="{{{{ message.tags }}}}">{{{{ message }}}}</li>
                    {{% endfor %}}
                </ul>
            {{% endif %}}

            <form action="{{% url 'predict_ai' %}}" method="post">
                {{% csrf_token %}}
                <textarea name="prompt">{{{{ default_prompt }}}}</textarea>
                <button type="submit">提交</button>
            </form>

            <p>更多資訊請訪問 <a href="{{% url 'about' %}}">關於我們</a>。</p>
        </body>
        </html>
        {{% endraw %}}
        ```text
    *   **Python 代碼中的模板字串 (僅為範例，不是直接渲染)**:
        ```python
        # Django 模板語法範例 (注意：此為字串，非直接模板渲染，用於演示其形式)
        template_string_example = """
        # {{% load i18n %}}
        # <h1>{{{{ user.get_full_name }}}}</h1>
        # {{% if user.is_authenticated %}}
        #     <p>{{% translate You are logged in. %}}</p>
        # {{% else %}}
        #     <p>{{% translate Please log in. %}}</p>
        # {{% endif %}}
        """
        ```text
    *   **行內提及模板語法**: 當在文本中提及時，請使用反引號，例如使用 `{{% url 'my_view' %}}` 來生成網址，或 `{{{{ object.field }}}}` 來顯示變數。

## 6. 維護和監控

部署完成後，持續的維護和監控對於確保 AI 服務的穩定、安全和高性能運行至關重要。

### 6.1 定期更新

*   **操作系統**: 定期運行 `sudo apt update && sudo apt upgrade -y` 以更新操作系統和已安裝的軟體包，修復安全漏洞。
*   **Python 依賴**: 定期檢查 `requirements.txt` 中的 Python 包是否有更新，並在開發/測試環境中測試後，再更新到生產環境。
    ```bash
    pip install --upgrade -r requirements.txt
    ```text
*   **AI 模型**: 如果您的 AI 模型有新版本或需要重新訓練以提高性能，請遵循您的模型更新流程進行部署。

### 6.2 日誌監控

*   **集中式日誌**: 考慮使用集中式日誌管理系統 (例如 ELK Stack: Elasticsearch, Logstash, Kibana 或 Loki, Grafana, Promtail) 來收集、存儲和分析來自 Nginx、Gunicorn 和應用程式的所有日誌。
*   **定期審查**: 定期審查日誌文件，尋找錯誤、警告、異常訪問模式或性能瓶頸。
    *   `sudo journalctl -u your_ai_app`
    *   `sudo tail -f /var/log/nginx/access.log`
    *   `sudo tail -f /var/log/nginx/error.log`

### 6.3 性能監控

*   **資源利用率**: 使用工具如 `htop`, `top`, `free -h` 來監控 CPU、記憶體和磁碟使用率，特別關注 AI 模型推斷時的資源消耗峰值。
*   **應用程式性能**: 部署應用程式性能監控 (APM) 工具 (如 Prometheus/Grafana, Sentry, New Relic, Datadog) 來監控請求響應時間、錯誤率、API 調用次數和數據庫查詢性能。
*   **AI 模型性能**: 監控 AI 模型的推斷時間、準確性（如果適用）和資源消耗，確保其在生產環境中表現良好。

### 6.4 備份策略

*   **數據庫備份**: 定期備份您的 PostgreSQL 數據庫。您可以使用 `pg_dump` 命令來執行此操作。
    ```bash
    pg_dump -U your_db_user your_ai_app_db > /path/to/backup/your_ai_app_db_$(date +%Y%m%d).sql
    ```text
*   **代碼和配置備份**: 定期備份您的應用程式代碼、Gunicorn 服務文件、Nginx 配置和 `settings.py` 等關鍵配置文件。由於代碼通常在 Git 倉庫中，確保您的生產分支與最新的穩定版本同步。

### 6.5 安全性

*   **防火牆**: 定期審查您的防火牆規則，確保只開放必要的端口。
*   **軟體漏洞**: 及時應用操作系統和所有軟體包的安全補丁。
*   **密碼和密鑰**: 使用強密碼，並定期更換。將敏感信息（如數據庫密碼、`SECRET_KEY`）存儲在環境變數或專用的配置管理工具中，而不是直接硬編碼在代碼裡。
*   **SSL/TLS**: 確保您的 SSL 證書始終有效且已正確續訂。Certbot 會自動處理 Let's Encrypt 證書的續訂。

通過遵循這些維護和監控的最佳實踐，您可以確保您的 Web-Based AI 服務在生產環境中保持穩定、安全和高效運行。