---
title: how-to-start-custom-vaultwarden
date: 2024-09-29 15:58:30
tags:
---

# 如何启动私有 Vaultwarden

`docker-compose.yml` 配置文件

```yaml
version: "3.9"
services:
 vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    restart: unless-stopped
    environment:
      - WEBSOCKET_ENABLED=true
      - SIGNUPS_ALLOWED=false
      - ADMIN_TOKEN=yourpassword
    volumes:
      - /var/vaultwarden:/data
    ports:
      - 127.0.0.1:8080:80
      - 3012:3012
```

# 反向代理

```nginx
server {
  listen 80;
  listen [::]:80;
  server_name local.example.com;
  return 301 https://$host$request_uri;
}

server {
  listen 443 ssl http2;
  listen [::]:443 ssl http2;
  server_name local.example.com;

  ssl_certificate /etc/letsencrypt/live/repo.example.com/fullchain.pem; # managed by Certbot
  ssl_certificate_key /etc/letsencrypt/live/repo.example.com/privkey.pem; # managed by Certbot


  # Allow large attachments
  client_max_body_size 128M;
  location / {
    proxy_pass http://localhost:8080;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
  }

  location /notifications/hub {
    proxy_pass http://localhost:3012;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
  }
    
  location /notifications/hub/negotiate {
    proxy_pass http://localhost:8080;
  }
}
```