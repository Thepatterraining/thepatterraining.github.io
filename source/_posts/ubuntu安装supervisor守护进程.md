---
title: ubuntu安装supervisor守护进程
date: 2020-12-09 10:12:47
tags: ['supervisor']
category: 小工具
article: ubuntu安装supervisor守护进程
---

# supervisor

supervisor是一个守护进程软件。

## 安装

ubuntu安装很简单。直接apt

```
sudo apt-get install supervisor
```

## 配置

修改配置项，加入我们的命令

```
sudo vim /etc/supervisor/conf.d/test.conf
```

复制下面的内容

```
[program:命令名称]
;目录
;directory=/test 这里我注释掉了，如果你加上以后报错可以注释掉试试
;执行的命令如果没有配置环境变量或者软连接请使用全路径
command=php /test/think queue:listen --queue im
process_name=%(program_name)s_%(process_num)02d;
numprocs = 3
autostart = true
startsecs = 5
autorestart = true
startretries = 3
;执行的linux用户
user=root
redirect_stderr = true
stdout_logfile_maxbytes = 50MB
stdout_logfile_backups = 20
;日志位置
stdout_logfile = /var/log/supervisor/queue_worker.log
loglevel=info
```

然后启动supervisor，因为是apt安装的，所以很好启动

```
sudo service supervisor start
```

查看启动了没有，有两种方法，一种status命令，还有一种查看linux进程

```
sudo service supervisor status
```

查看进程

```
ps -aux |grep super
```

再查看我们的命令挂起来没有。我们执行的php命令所以直接查看php

```
ps -aux|grep php
```
如果看见刚刚的命令就成功了。

