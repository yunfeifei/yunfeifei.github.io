---
layout: default
title:  "Docker部署zabbix3.2监控系统"
date:   2017-03-13 23:50:00
categories: main
---

## 官方文档
官方文档参考地址：[Installation from containers](https://www.zabbix.com/documentation/3.2/manual/installation/containers)

## 镜像简介

Docker上面的Zabbix的镜像分为三个部分：

* 数据库
* 服务器
* Web界面

所以，我们要部署一个完整的zabbix，至少需要三个镜像。

这里我们数据库基于mysql、web服务器基于Nginx。所以，我们的镜像选择如下：

* zabbix server镜像：zabbix/zabbix-server-mysql
* zabbix web interface镜像：zabbix/zabbix-web-nginx-mysql
* 数据库镜像：mysql

## 启动容器

1. 启动mysql容器

mysql容器的参数如下：

```bash
docker run --name mysql-server -t \
    -e MYSQL_DATABASE="zabbix" \
    -e MYSQL_USER="zabbix" \
    -e MYSQL_PASSWORD="zabbix_pwd" \
    -e MYSQL_ROOT_PASSWORD="root_pwd" \
    -d mysql:5.7
```

这里我们只指定mysql的密码来启动：

```bash
# docker run -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=root -d mysql
```

2. 启动zabbix-server-mysql 容器

```bash
docker run --name zabbix-server-mysql -t \
    -e DB_SERVER_HOST="mysql-server" \
    -e MYSQL_DATABASE="zabbix" \
    -e MYSQL_USER="zabbix" \
    -e MYSQL_PASSWORD="zabbix_pwd" \
    -e MYSQL_ROOT_PASSWORD="root_pwd" \
    -e ZBX_JAVAGATEWAY="zabbix-java-gateway" \
    --link mysql-server:mysql \
    --link zabbix-java-gateway:zabbix-java-gateway \
    -p 10051:10051 \
    -d zabbix/zabbix-server-mysql:latest
```

这里我们用以下命令启动：

```bash
docker run --name zabbix-server-mysql -t \
    -e DB_SERVER_HOST="mysql" \
    -e MYSQL_DATABASE="zabbix" \
    -e MYSQL_USER="root" \
    -e MYSQL_PASSWORD="root" \
    -e MYSQL_ROOT_PASSWORD="root" \
    --link mysql:mysql \
    -p 10051:10051 \
    -d zabbix/zabbix-server-mysql
```

3. 启动zabbix-web-nginx-mysql容器

```bash
docker run --name zabbix-web-nginx-mysql -t \
    -e DB_SERVER_HOST="mysql-server" \
    -e MYSQL_DATABASE="zabbix" \
    -e MYSQL_USER="zabbix" \
    -e MYSQL_PASSWORD="zabbix_pwd" \
    -e MYSQL_ROOT_PASSWORD="root_pwd" \
    --link mysql-server:mysql \
    --link zabbix-server-mysql:zabbix-server \
    -p 80:80 \
    -d zabbix/zabbix-web-nginx-mysql:latest
```

这里我们用以下命令启动：

```bash

docker run --name zabbix-web-nginx -d 
--link mysql:mysql 
--link zabbix-server-mysql:zabbix-server-mysql 
-p 80:80 -e DB_SERVER_HOST="mysql" 
-e MYSQL_USER="root" -e MYSQL_PASSWORD="root" zabbix/zabbix-web-nginx-mysql 

```

## Docker的调试

可以用docker logs [容器名称] 来查看日志或错误信息。