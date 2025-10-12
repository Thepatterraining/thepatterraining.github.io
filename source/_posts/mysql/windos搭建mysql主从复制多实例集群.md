---
title: windos搭建mysql主从复制多实例集群
date: 2025-08-04 10:12:47
tags: ['数据库','数据库原理','sql','mysql']
category: mysql
article: windos搭建mysql主从复制多实例集群
---

> 大家好，我是大头，职高毕业，现在大厂资深开发，前上市公司架构师，管理过10人团队！
> 我将持续分享成体系的知识以及我自身的转码经验、面试经验、架构技术分享、AI技术分享等！
> 愿景是带领更多人完成破局、打破信息差！我自身知道走到现在是如何艰难，因此让以后的人少走弯路！
> 无论你是统本CS专业出身、专科出身、还是我和一样职高毕业等。都可以跟着我学习，一起成长！一起涨工资挣钱！
> 关注我一起挣大钱！文末有惊喜哦！

> 关注我发送“MySQL知识图谱”领取完整的MySQL学习路线。
> 发送“电子书”即可领取价值上千的电子书资源。
> 发送“大厂内推”即可获取京东、美团等大厂内推信息，祝你获得高薪职位。
> 发送“AI”即可领取AI学习资料。

# MySQL零基础教程

本教程为零基础教程，零基础小白也可以直接学习，有基础的可以跳到后面的原理篇学习。
基础概念和SQL已经更新完成。

接下来是应用篇，应用篇的内容大致如下图所示。

![应用学习](https://thepatterraining.github.io/images/mysql/mysql1-2.png)
 

## windos搭建mysql主从复制多实例集群

因为没有更多的机器来搭建多机器的集群环境，所以这里使用单机多实例监听多端口来实现mysql的集群环境。

### 为什么要使用mysql主从环境

一般来说读流量大于写流量，所以当流量过大时可以通过mysql主从来扩展从库，读取从库从而应对读流量的冲击。主从复制的实现原理和优缺点有兴趣的可以看我这个文章[mysql主从读写分离](https://blog.csdn.net/Thepatterraining/article/details/105248917)。

### 搭建主从环境

这里假设你已经下载安装好mysql了。已经可以启动一个mysql 3306的实例了。

现在我们复制一个配置文件。我原来的配置文件是`my.ini`，我直接复制粘贴改个名字，为了省事命名成`3307my.ini`。

> cp my.ini 3307my.ini

接下来我们有了两个配置文件，一个`my.ini`，一个`3307my.ini`。

我们修改`3307my.ini`的配置文件内容。

```conf
[mysqld]
port=3307  # 新实例的端口
basedir=D:/MySQL5.7.26/3307  # 新实例的文件夹 我直接放在了原来mysql文件夹下面
datadir=D:/MySQL5.7.26/3307/data/ # 新实例的数据存放路径
character-set-server=utf8
default-storage-engine=InnoDB
max_connections=100
collation-server=utf8_unicode_ci
init_connect='SET NAMES utf8'
innodb_buffer_pool_size=64M
innodb_flush_log_at_trx_commit=1
innodb_lock_wait_timeout=120
innodb_log_buffer_size=4M
innodb_log_file_size=256M
interactive_timeout=120
join_buffer_size=2M
key_buffer_size=32M
log_error_verbosity=1
max_allowed_packet=16M
max_heap_table_size=64M
myisam_max_sort_file_size=64G
myisam_sort_buffer_size=32M
read_buffer_size=512kb
read_rnd_buffer_size=4M
server_id=2  # server_id需要修改成和原来不一样的
skip-external-locking=on
sort_buffer_size=256kb
table_open_cache=256
thread_cache_size=16
tmp_table_size=64M
wait_timeout=120

log-bin=D:/MySQL5.7.26/3307/mysql-bin  # 开启binlog binlog用于主从同步

server-id=123455 # server-id同上

log-error=D:/MySQL5.7.26/3307_err.log              #错误日志
```

修改完成接下来使用这个配置文件启动，你会发现启动失败，哈哈哈哈。

因为新的数据库没有`mysql`的基本信息那些库和表还有插件，所以我们需要复制过去，我是采用了这种简单粗暴的方法，网上还有使用`mysql_install_db`的，不过我发现我的mysql没有这玩意。

我们复制`data`和`share`两个文件夹复制到`3307`文件夹里面。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/37fb8b9c000993be9dd50e548a25ca61.png)

下面是`./3307/`文件夹下面的内容，复制过来以后这两个文件夹就过来了。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/28725bcf0f79dd845d2ae0cb1858d7a4.png)
接下来你可以启动第二个`mysql`实例了。回到`3307`目录的上层目录，也就是你的mysql根目录。然后运行下面的命令。使用刚才的`3307my.ini`启动新实例。

> ./bin/mysqld.exe --defaults-file=./3307my.ini

启动成功后可以进行连接了。连接上去以后可以看到现在和`3306`库一样了。接下来开始配置`主从复制`。

我们先进入`3306`主库客户端，查看一些数据。执行下面的命令。
```sql
show master status;
```

可以看到下面的内容。下面的`file`参数和`position`参数要在从库配置中用到。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9c1ce045d84ac27cc797a09cb3a3aac9.png)

接下里进入`3307`客户端执行下面的命令。

```sql
change master to 
master_host='127.0.0.1',  --master地址
master_port=3306,         --master端口
master_user='root',       --master登录用户名
master_password='pa88word', --master登录密码
master_log_file='mysql-bin.000009',  --要开始同步的binlog文件 上面的file参数
master_log_pos=154;                  --要开始同步的具体指针位置 上面的position参数
```

执行完这个命令，从库就知道主库在哪了，就知道谁是主库了，也知道该从哪里开始同步了。

你以为接下来就完了？不不不，接下来还需要启动从库才行。

> start slave;

这个命令是什么呢，主要是启动从库的更新线程，从库有两个更新线程，一个io线程负责把主库的`binlog`写入从库的`relay log`。一个sql线程负责把`relay log`中的信息写入从库。

接下里看看从库的状态，执行下面的命令。

>show slave status;

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ba74e8ee0b04c2ffc90a13a8fdfa286a.png)

这里面主要看`Slave_IO_Running`和`Slave_SQL_Running`也就是上面说的两个线程是否在运行，如果是`Yes`，那么就ok了。

如果不是`Yes`可以退出重启一下mysql服务试试。

如果已经可以了，那么可以在`3306`主库里面修改一些数据进行测试了。如果有问题你没搭建成功，欢迎你来找我。下一篇文章教你搭建 mysql 2主多从集群。

## 文末福利

> 关注我发送“MySQL知识图谱”领取完整的MySQL学习路线。
> 发送“电子书”即可领取价值上千的电子书资源。
> 发送“大厂内推”即可获取京东、美团等大厂内推信息，祝你获得高薪职位。
> 发送“AI”即可领取AI学习资料。
> 部分电子书如图所示。

![](https://thepatterraining.github.io/images/bottom1.png)

![](https://thepatterraining.github.io/images/bottom2.png)

![](https://thepatterraining.github.io/images/bottom3.png)

![](https://thepatterraining.github.io/images/bottom4.png)

