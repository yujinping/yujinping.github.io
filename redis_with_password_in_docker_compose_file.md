+++
title="在docker-compose中启动redis并设置密码"
tags=["docker","mac"]
categories=["tools"]
date="2022-06-07T12:32:00+08:00"
toc=true
+++

## docker-compose.yml

```yaml
  redis:
    image: "redis:7.0.0-alpine"
    container_name: "redis"
    privileged: true
    ports:
      # docker容器redis默认端口号:6379
      - "6379:6379"
    command:
      - redis-server
      - --requirepass
      - "your-password-here"
```

## 命令行方式

启动 docker 容器时携带参数 `--requirepass your-password` 即可
