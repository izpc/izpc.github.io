---
title: 如何搭建私有仓库
date: 2024-09-29 15:11:40
tags:
- nexus
- pip
- npm
- docker
- helm
---

# 如何搭建私有仓库

Nexus Repository Manager 是一个强大的仓库管理器，支持多种包管理器，如 Maven、npm、Docker、Helm、PyPI 等。

在本文中，我们将介绍如何使用 Nexus Repository Manager 搭建私有仓库，以便于管理和分享软件包。

`docker-compose.yml`

```yaml
version: '2.0'

services:
  nexus:
    image: sonatype/nexus3
    container_name: nexus
    restart: always
    ports:
      - "127.0.0.1:8081:8081"
      - "127.0.0.1:5000:5000"
    volumes:
      - ${HOME}/nexus-data:/nexus-data
```


## 反向代理

[https://help.sonatype.com/en/run-behind-a-reverse-proxy.html](https://help.sonatype.com/en/run-behind-a-reverse-proxy.html)

`repo.example.com` 

```nginx
server {
    if ($host = repo.example.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot



    listen 80;
        server_name repo.example.com;
        # return 301 https://$host$request_uri;

        # allow large uploads of files - refer to nginx documentation
        client_max_body_size 1G;

        # optimize downloading files larger than 1G - refer to nginx doc before adjusting
        #proxy_max_temp_file_size 2G;

    location / {

            proxy_pass http://localhost:8080/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }


}

server {

    listen 443 ssl;
        server_name repo.example.com;

        # allow large uploads of files - refer to nginx documentation
        client_max_body_size 1G;

        # optimize downloading files larger than 1G - refer to nginx doc before adjusting
        #proxy_max_temp_file_size 2G;
    ssl_certificate /etc/letsencrypt/live/repo.example.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/repo.example.com/privkey.pem; # managed by Certbot
        include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
        ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

        location / {

            proxy_pass http://localhost:8081/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto "https";
        }


}

```


## Docker Registry

`docker.example.com`

```nginx
server {
    if ($host = docker.example.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot



    listen 80;
        server_name docker.example.com;
        # return 301 https://$host$request_uri;

        # allow large uploads of files - refer to nginx documentation
        client_max_body_size 10G;

        # optimize downloading files larger than 1G - refer to nginx doc before adjusting
        #proxy_max_temp_file_size 2G;

    location / {

            proxy_pass http://localhost:5000/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }


}

server {

    listen 443 ssl;
        server_name docker.example.com;

        # allow large uploads of files - refer to nginx documentation
        client_max_body_size 10G;

        # optimize downloading files larger than 1G - refer to nginx doc before adjusting
        #proxy_max_temp_file_size 2G;
    ssl_certificate /etc/letsencrypt/live/repo.example.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/repo.example.com/privkey.pem; # managed by Certbot
        include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
        ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

        location / {

            proxy_pass http://localhost:5000/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto "https";
        }


}
```


## Helm Repository

[https://sonatype.github.io/helm3-charts/](https://sonatype.github.io/helm3-charts/)