## Nginx

- [安裝](#安裝)
- [反向代理](#反向代理)
- [配置證書](#配置證書)

### 安裝

```sh
sudo apt update
sudo apt install nginx
```

檢查是否正常運作

```sh
sudo systemctl status nginx
```

### 反向代理

1. 建立設定檔

```sh
sudo nano /etc/nginx/sites-available/{域名}
```

2. 寫入設定

```
server {
    listen 80;
    server_name {域名};

    location / {
        proxy_pass http://localhost:22222; // 代理 URL
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

3. 啟用設置

```sh
sudo ln -s /etc/nginx/sites-available/{域名} /etc/nginx/sites-enabled/
```

4. 測試語法

```sh
sudo nginx -t
```

5. 重新加載以套用新設定（必須先通過語法測試）

```sh
sudo systemctl reload nginx
```

### 配置證書

> 須先完成 http-01 挑戰

```sh
sudo certbot --nginx -d {域名}
```
