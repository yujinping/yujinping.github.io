+++
title="使用docker alpine 的 java容器运行grpc服务报 \"Could not find TLS ALPN provider; no working netty-tcnative, Conscrypt, or Jetty NPN/ALPN available\"的错误"
tags=["docker","alpine","grpc"]
categories=["docker"]
date="2023-03-06T10:23:00+08:00"
toc=true
+++

## 背景

在使用 java8（amazon corretto）本机（mac book pro）编译和运行项目时，无任何异常。将编译后的 jar 文件打包到 docker 镜像后，再运行，报”Could not find TLS ALPN provider; no working netty-tcnative, Conscrypt, or Jetty NPN/ALPN available“ 这个错误。

Dockerfile 内容如下：

```dockerfile
FROM amazoncorretto:8-alpine
RUN sed -i 's/dl-cdn.alpinelinux.org/mirror.tuna.tsinghua.edu.cn/g' /etc/apk/repositories

RUN apk --update add curl bash ttf-dejavu && \
rm -rf /var/cache/apk/*

RUN apk add -U tzdata && \
/bin/cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo 'Asia/Shanghai' > /etc/timezone

ARG APP_NAME
ENV APP_NAME=${APP_NAME}
ENV JAVA_X_PARAM=${JAVA_X_PARAM:-'-Xmx1024m -Xms256m'}
ENV NACOS_HOST=${NACOS_HOST}
ENV NACOS_PORT=${NACOS_PORT:-'8848'}
ENV NACOS_NAMESPACE=${NACOS_NAMESPACE:-'test'}
ENV NACOS_GROUP=${NACOS_GROUP:-'TEST_GROUP'}
ENV SPRING_PROFILE=${SPRING_PROFILE:-'test'}
ENV LANG=zh_CN.UTF-8
ENV LANGUAGE=zh_CN.UTF-8

WORKDIR /opt/app

COPY ${APP_NAME}.jar ./${APP_NAME}.jar

CMD java \
-Drun.nacos.host=${NACOS_HOST} \
-Drun.nacos.port=${NACOS_PORT} \
-Drun.nacos.namespace=${NACOS_NAMESPACE} \
-Drun.spring.profile=${SPRING_PROFILE} \
-Drun.service.group=${NACOS_GROUP} \
-jar ./${APP_NAME}.jar
```

## 分析及解决

鉴于本机运行正常，相同的 java 版本。不同的是本机操作系统是 macos,而 docker 容器是 alpine linux. 因此可能是操作系统兼容性的原因导致的。
于是通过搜索，在 https://stackoverflow.com/questions/54733389/could-not-find-tls-alpn-provider-no-working-netty-tcnative-conscrypt-or-jetty 找到问题。

根源在于 alpine Linux 对于 grpc 协议的兼容性较为缺乏。需要额外安装支持。

根据文中的建议，增加

```bash
RUN apk add gcompat
ENV LD_PRELOAD=/lib/libgcompat.so.0
```

问题得到解决。
修正后完整的 Dockerfile 内容如下:

```dockerfile
FROM amazoncorretto:8-alpine
RUN sed -i 's/dl-cdn.alpinelinux.org/mirror.tuna.tsinghua.edu.cn/g' /etc/apk/repositories

# gcomat 用于对grpc的支持，包括rocketmq等,参见: https://stackoverflow.com/questions/54733389/could-not-find-tls-alpn-provider-no-working-netty-tcnative-conscrypt-or-jetty

RUN apk --update add curl bash ttf-dejavu gcompat && \
rm -rf /var/cache/apk/*

RUN apk add -U tzdata && \
/bin/cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo 'Asia/Shanghai' > /etc/timezone

ENV LD_PRELOAD=/lib/libgcompat.so.0

ARG APP_NAME
ENV APP_NAME=${APP_NAME}
ENV JAVA_X_PARAM=${JAVA_X_PARAM:-'-Xmx1024m -Xms256m'}
ENV NACOS_HOST=${NACOS_HOST}
ENV NACOS_PORT=${NACOS_PORT:-'8848'}
ENV NACOS_NAMESPACE=${NACOS_NAMESPACE:-'test'}
ENV NACOS_GROUP=${NACOS_GROUP:-'TEST_GROUP'}
ENV SPRING_PROFILE=${SPRING_PROFILE:-'test'}
ENV LANG=zh_CN.UTF-8
ENV LANGUAGE=zh_CN.UTF-8
ENV LD_PRELOAD=/lib/libgcompat.so.0

WORKDIR /opt/app

COPY ${APP_NAME}.jar ./${APP_NAME}.jar

CMD java \
-Drun.nacos.host=${NACOS_HOST} \
-Drun.nacos.port=${NACOS_PORT} \
-Drun.nacos.namespace=${NACOS_NAMESPACE} \
-Drun.spring.profile=${SPRING_PROFILE} \
-Drun.service.group=${NACOS_GROUP} \
-jar ./${APP_NAME}.jar
```
