+++
title="批量删除docker的容器及镜像"
tags=["freemarker","java"]
categories=["java"]
date="2023-03-06T17:23:00+08:00"
toc=true
+++

## 批量删除所有容器

```bash
docker rm `docker ps -a -q`
```

## 批量删除所有镜像

```bash
docker rmi `docker images -q`
```

## 批量删除含有关键字的容器

```bash
docker rm `docker ps -a | grep dmj | awk '{print $1}'`
```

## 批量删除含有关键字的镜像

```bash
docker rmi --force `docker images | grep dmj | awk '{print $1}'`
```

## 关键知识点

- `docker ps -a -q` 命令输出容器 id
- 使用`grep "关键字"` 命令查找想要的容器或镜像
- 使用 `awk '{print $1}'` 命令将第一列输出打印出来
- 使用 `|` 管道符将各个命令串接起来最后得到想要的结果
