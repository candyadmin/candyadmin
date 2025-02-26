---
title: nginx配置https
date: 2025-02-26 19:52:07
categories: linux
tag: nginx
---
```shell
server {
    listen 80;
    server_name xxx.beeflyx.cloud;

    # 强制 HTTP 重定向到 HTTPS
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name xxx.beeflyx.cloud;

    ssl_certificate /etc/nginx/conf.d/cert/xxx.beeflyx.cloud_bundle.crt;
    ssl_certificate_key /etc/nginx/conf.d/cert/xxx.beeflyx.cloud.key;

    location / {
        proxy_pass http://127.0.0.1:3000;
    }
}
```