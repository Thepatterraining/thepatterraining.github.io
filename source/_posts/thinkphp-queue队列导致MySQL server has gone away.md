---
title: thinkphp-queue队列导致MySQL server has gone away
date: 2020-08-25 10:12:47
tags: ['php','thinkphp','queue']
category: php
article: thinkphp-queue队列导致MySQL server has gone away
---

# thinkphp-queue队列导致MySQL server has gone away

虽然队列一时爽，不过还是有缺点的，比如当队列运行时间长了会报错 MySQL server has gone away

原因是使用`work`模式运行时间长了以后没有释放mysql数据库的链接，导致时间长了以后被mysql server端判断超时切断了链接。

可以改用`listen`模式运行，这样每次都是启动一个新的work进程来运行程序，每次都会新链接数据库。

可以使用tp的断线重连功能。修改配置文件`config/database.php`

```php

// 数据库连接配置信息
    'connections'     => [
        'mysql' => [
            // 是否需要断线重连
            'break_reconnect'   => true,
        ],
```
