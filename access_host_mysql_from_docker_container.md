---
title: 如何在docker内部连接宿主机的MYSQL
tags:
  - docker
  - mysql
categories:
  - docker
date: 2022-04-24T10:10:00
toc: true
---

## windows/mac 平台连接方式

windows 和 mac 都是通过`host.docker.internal`去连接宿主机。

当你启动了一个 docker web 服务，需要修改 docker web 使用的 mysql 地址，将 `localhost` 改成 `host.docker.internal` 即可。

## linux 平台连接方式

通过获取 docker 内部的网关获取到宿主机的 ip：

```sh
docker inspect <container-id-or-name> | grep Gateway
"Gateway": "",
"IPv6Gateway": "",
"Gateway": "172.18.0.1",
"IPv6Gateway": "",
```

对于本例中 docker 应用程序使用的 MySQL 指向宿主机的 `172.18.0.1:3306`
