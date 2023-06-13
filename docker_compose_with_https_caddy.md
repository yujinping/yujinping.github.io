---
title: 使用docker+caddy搭建https站点的compose配置
tags:
  - docker
  - caddy
categories:
  - docker
date: 2022-05-07T10:10:00
toc: true
---

```yaml
version: "3.7"
services:
  # http/2 server
  caddy:
    image: caddy:2.4.6-alpine
    container_name: caddy
    restart: unless-stopped
    network_mode: "host"
    environment:
      - TZ=Asia/Shanghai
      - agree
      - email=87418455@qq.com
    volumes:
      - "~/docker/caddy/conf/Caddyfile:/etc/Caddyfile"
      - "~/docker/caddy/.caddy:/root/.caddy"
      - "~/docker/caddy/logs:/opt/logs"
      - "~/docker/caddy/www:/opt/www"
    ports:
      - 80:80
      - 443:443
```
