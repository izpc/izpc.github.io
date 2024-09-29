---
title: how-to-start-database-proxy
date: 2024-09-29 15:34:32
tags:
---

```nginx
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

worker_rlimit_nofile 65535;

events {
    worker_connections 10240;
    # multi_accept on;
}

http {

        allow all;

    ##
    # Basic Settings
    ##

        ## Start nexus https://help.sonatype.com/repomanager3/planning-your-implementation/run-behind-a-reverse-proxy
        proxy_send_timeout 120;
        proxy_read_timeout 300;
        proxy_buffering    off;
        proxy_request_buffering off;
        #keepalive_timeout  5 5;
        #tcp_nodelay        on;
        ## End nexus

    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    # server_tokens off;

    # server_names_hash_bucket_size 64;
    # server_name_in_redirect off;

    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    ##
    # SSL Settings
    ##

    ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
    ssl_prefer_server_ciphers on;

    ##
    # Logging Settings
    ##

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    ##
    # Gzip Settings
    ##

    gzip on;

    # gzip_vary on;
    # gzip_proxied any;
    # gzip_comp_level 6;
    # gzip_buffers 16 8k;
    # gzip_http_version 1.1;
    # gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

    ##
    # Virtual Host Configs
    ##

    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}


stream {
    log_format proxy '$remote_addr [$time_local] '
                 '$protocol $status $bytes_sent $bytes_received '
                 '$session_time "$upstream_addr" '
                 '"$upstream_bytes_sent" "$upstream_bytes_received" "$upstream_connect_time"';

    access_log /var/log/nginx/tcp-access.log proxy ;
    open_log_file_cache off;



    # mysql: 3306
    server {
        listen 33078;
        proxy_pass your.domain.com:3306;
        proxy_timeout 300m;
    }

}

```