---
title: 基于docker快速搭建多平台laravel环境-laradock
date: 2020-04-05 14:48:27
tags: ['docker','laravel', 'php']
category: php
article: 基于docker快速搭建多平台laravel环境-laradock
---

# 基于docker快速搭建多平台laravel环境-laradock

现在docker技术越来越火，docker的应用也越来越多。

我们为什么要用docker呢，因为它能提供你一个纯净的环境，能统一所有开发人员的环境，公司的技术有很多人，那每个人装的环境都可能不一样，你是php7.3，他是php7.0，你是mysql8.0，他是mysql5.6，这些环境上的差异有时候会导致代码的错误。

还有环境这东西装一次就够用了，你家里的电脑环境和公司的环境也有可能不一致。使用docker装环境之后，我们可以装完之后打包起来，在任何一个docker上运行这个配置文件都可以生成相同的环境。

## 下载安装

>  git clone https://github.com/Laradock/laradock.git

<!--more-->

把这个库克隆下来以后，进入目录

> cd [your work]/laradock

复制env-example文件到.env文件

> cp env-example .env

修改.env配置

```
DB_HOST=mysql
REDIS_HOST=redis
```

在.env中配置工作目录,如果你的目录wwwroot和laradock目录同级则这样设置

```
APP_CODE_PATH_HOST=../wwwroot
```

配置nginx文件，打开 ./nginx/sites/default.conf，修改下面两行

`/var/www`目录相当于上面配置的../wwwroot目录

```
server_name your server name;
root /var/www/[your dir]/public;
```

## 使用

启动docker 程序

> docker-compose up -d nginx redis mysql

如果启动中报错

> ERROR: Service 'mysql' failed to build: Get https://registry-1.docker.io/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)

报这种错误，那么你需要一个docker镜像加速器了，可以使用daocloud的加速器。

> https://www.daocloud.io/mirror#accelerator-doc

按照配置添加完以后再次运行命令就可以了

## .env文件参数介绍

PHP_VERSION=7.3 这个参数是php的版本，默认php7.3

PHALCON_VERSION=3.4.5 phalcon的版本

PHP_INTERPRETER=php-fpm php解释器，默认php-fpm

nginx 配置

NGINX_HOST_HTTP_PORT=80  //nginx http 端口
NGINX_HOST_HTTPS_PORT=443 //nginx https 端口
NGINX_HOST_LOG_PATH=./logs/nginx/  //nginx 日志
NGINX_SITES_PATH=./nginx/sites/   //nginx 配置
NGINX_PHP_UPSTREAM_CONTAINER=php-fpm //nginx 使用php-fpm连接php
NGINX_PHP_UPSTREAM_PORT=9000  //php端口
NGINX_SSL_PATH=./nginx/ssl/ //ssl

mysql配置

MYSQL_VERSION=5.7    mysql版本 默认是8.0 建议修改成5.7
MYSQL_DATABASE=default  数据库
MYSQL_USER=default      用户名
MYSQL_PASSWORD=secret   密码
MYSQL_PORT=3306         mysql 端口
MYSQL_ROOT_PASSWORD=root mysql root 密码
MYSQL_ENTRYPOINT_INITDB=./mysql/docker-entrypoint-initdb.d

REDIS_PORT=6379 redis端口