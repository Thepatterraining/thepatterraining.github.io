---
title: window php 环境搭建一
date: 2018-03-15
tags: [php, nginx, mysql, redis, mongo, rabbitMq]
category: php
article: "php环境搭建一"
---

# 前言

### 开始写博客了，激动ing--此处省略一万字……

# 正文

直接说环境吧，每次到一个公司，配置环境是最烦的，嗯，博主比较菜，这个配置完n久都不在用的，一会就忘了，所以，你懂的。。环境清单如下：

    php
    nginx
    mysql
    redis
    mongo
    rabbitMq队列

下载 没什么好说的，百度官网，直接下载就行了，我相信大家都知道。

`php` 下载之后 直接复制 一个 php.ini-development 或者 php.ini-production 然后改名成 php.ini，你们猜的没错，这就是我们的配置啦。扩展之类的 从 Pecl下载好 ，然后在配置里面打开就好了。
下载 nginx 同上，一样官网下载，然后在你下载之后，里面有 conf 文件夹 里面的 nginx.conf 就是 nginx 的配置文件了。暂时什么都不用改，启动 nginx 然后访问 localhost 就会出现 nginx 的欢迎页面了。

`mysql` 下载是一样的，在安装的时候 要注意选择需要的东西安装就好了，博主只用了server and bench。安装的时候会设置 root 用户的密码。

`redis` 同样官网下载，之后咋们主要看 2个东西 redis-server.exe and redis-cli.exe ，一个服务器一个客户端。然后还有一个隐藏小boss，redis.windows.conf 启动 redis 服务的时候要带上这玩意，所以我们就要用 cmd 了， 打开 cmd 输入如下：

>路径\redis-server.exe redis.windows.conf

好了，然后点击 `redis-cli.exe` 就可以用了。完美。

`mongo` 同上，下载完之后 在 `bin` 文件夹里面 有 `mongod.exe` and `mongo.exe` 也是 服务器和客户端，对于 `mongo` 我们要建一个 `data\db` 这两个文件夹

最后到了 `rabbitMq` 了, 安装这个之前，要先安装 `erlang` ，直接下载安装， 然后就可以安装 `rabbitMq` 了，然后 我们 安装 `rabbitMq-plugins` 打开 cmd 输入

>路径\rabbitMq_server\sbin\rabbitmq-plugins enable rabbitmq_management

然后就会安装成功了。

说一下问题。
首先这些 环境变量 一定要弄上。
然后 `rabbitmq` 要先启动 在安装 `rabbitmq-plugins` ，我也忘了是不是了，你们根据情况来吧。

>安装完之后 进入浏览器 localhost:15672 就能访问管理页面了。


哈哈，其实还是有一些坑的，只不过写的时候想不起来了，所以这篇没什么干货。。

# 后记

>想起来一个东西，所有的命令输入 绝对路径 或者 进入相应文件夹 用 .\*.exe 的命令来执行

如果有什么想问的 欢迎加博主的qq and 微信


        