---
title: how-to-start-private-jenkins
date: 2024-09-29 15:22:18
tags:
- jenkins
- docker
---

# 如何启动私有 Jenkins

`docker-compose.yml` 配置文件

```yaml
version: '3'
services:
  jenkins:
    image: docker.foobar.com/jenkins
    user: root
    build: .
    restart: always
    container_name: jenkins
    extra_hosts:
      - "bitbucket.foobar.com:127.0.0.1"
    ports:
      - "127.0.0.1:8080:8080"
      - "127.0.0.1:50000:50000"
    volumes:
      - ${HOME}/jenkins/jenkins_home:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock
      - ${HOME}/.docker:/root/.docker
      - ${HOME}/.ssh:/root/.ssh
      #- /usr/bin/docker:/usr/bin/docker
      #- /usr/bin/docker-compose:/usr/bin/docker-compose
    environment:
      #- JENKINS_OPTS="--sessionTimeout=0"
      - JENKINS_OPTS=--sessionTimeout=43200 --sessionEviction=604800
```


# 反向代理

```nginx

upstream jenkins {
  keepalive 32; # keepalive connections
  server 127.0.0.1:8080; # jenkins ip and port
}

# Required for Jenkins websocket agents
map $http_upgrade $connection_upgrade {
  default upgrade;
  '' close;
}

server {       # Listen on port 80 for IPv4 requests

  server_name     deploy.example.com local.example.com;  # replace 'jenkins.example.com' with your server domain name

  # this is the jenkins web root directory
  # (mentioned in the output of "systemctl cat jenkins")
  access_log      /var/log/nginx/jenkins.access.log;
  error_log       /var/log/nginx/jenkins.error.log;

  # pass through headers from Jenkins that Nginx considers invalid
  ignore_invalid_headers off;

  location ~ "^/static/[0-9a-fA-F]{8}\/(.*)$" {
    # rewrite all static files into requests to the root
    # E.g /static/12345678/css/something.css will become /css/something.css
    rewrite "^/static/[0-9a-fA-F]{8}\/(.*)" /$1 last;
  }

  location /userContent {
    # have nginx handle all the static requests to userContent folder
    # note : This is the $JENKINS_HOME dir
    root /var/lib/jenkins/;
    if (!-f $request_filename){
      # this file does not exist, might be a directory or a /**view** url
      rewrite (.*) /$1 last;
      break;
    }
    sendfile on;
  }
  location / {
      # allow 1.1.1.1; # allow ip address to access the site
      # deny all;

      sendfile off;
      proxy_pass         http://jenkins;
      proxy_redirect     default;
      proxy_http_version 1.1;
      # Required for Jenkins websocket agents
      proxy_set_header   Connection        $connection_upgrade;
      proxy_set_header   Upgrade           $http_upgrade;
      proxy_set_header   Host              $host;
      proxy_set_header   X-Real-IP         $remote_addr;
      proxy_set_header   X-Forwarded-For   $proxy_add_x_forwarded_for;
      proxy_set_header   X-Forwarded-Proto $scheme;
      proxy_max_temp_file_size 0;
      #this is the maximum upload size
      client_max_body_size       10m;
      client_body_buffer_size    128k;
      proxy_connect_timeout      90;
      proxy_send_timeout         90;
      proxy_read_timeout         90;
      proxy_buffering            off;
      proxy_request_buffering    off; # Required for HTTP CLI commands
      proxy_set_header Connection ""; # Clear for keepalive
  }

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/repo.example.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/repo.example.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot


}


server {
    if ($host = deploy.example.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot

    if ($host = local.example.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


  listen          80;

  server_name     deploy.example.com local.example.com;
    return 404; # managed by Certbot


}
```