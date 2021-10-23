---
title: nginx二 -- 初识nginx
date: 2021-10-08 10:12:47
tags: ['nginx']
category: nginx
article: nginx二 -- 初识nginx
---

# nginx

## nginx的场景

三个主要场景
- 静态资源：通过本地文件系统提供服务
- 反向代理：nginx的强大性能，缓存，负载均衡
- API服务：openresty

## nginx的优点

- 高并发，高性能
- 可扩展性好
- 高可靠性
- 热部署
- BSD许可证


## nginx的组成

- nginx二进制可执行文件：由各模块原码编译出的一个文件
- nginx.conf:控制nginx行为
- access.log
- error.log

## vim nginx语法高亮

如果想要vim操作nginx的时候有语法高亮，那么只需要把nginx文件夹带的vim配置文件复制到自己的vim配置文件下面就可以了

找到nginx源码目录，注意这里是源码目录，下载的tar包解压出来的目录，而不是编译安装后的目录，下面有一个`contrib`文件夹，这个文件夹里面有`vim`文件夹，把这个里面的配置文件都复制一下就好了

```
cp -r nginx/contrib/vim/ ~/.vim/
```

## 已安装的nginx 增加模块

首先到达源码目录`cd /usr/local/src/nginx1.19`

然后开始编译

- --prefix 参数代表你的nginx安装目录，如果是已安装的nginx，那么这个目录就选择已安装的目录，没有安装的可以随便选想安装到的目录
- --with 开头的代表你想开启的模块

```
sudo ./configure --with-file-aio --with-http_ssl_module --with-http_v2_module --with-http_realip_module --prefix=/usr/local/src/nginx
```

然后执行`make`

```
sudo make
```

接下来就可以复制nginx源文件了，这里只执行make，不需要执行make install

现在假设你已经在nginx源码目录了

```
cd ./objs  //移动到objs目录下
cp nginx /usr/local/src/nginx/sbin/    //把objs目录下的nginx复制到你原来的nginx安装目录下面的sbin目录替换原来的nginx源文件
```

如果是新安装，那么不需要复制，直接执行 sudo make install 就可以了


## 配置语法

- 配置文件由指令和指令块构成
- 每条指令以;分号结尾，指令与参数间用空格分隔
- 指令块以{}大括号将多条指令组织在一起
- include 语句允许组合多个配置文件以提升可维护性
- 使用 #井号添加注释，提高可读性
- 使用 $符号作为变量
- 部分指令的参数支持正则表达式


## http配置的指令块

- http
- server
- upstream
- location


## nginx 命令行

- 帮助 -h
- 指定配置文件 -c
- 指定配置指令 -g
- 指定运行目录 -p
- 发送信号 -s
- 测试配置文件语法 -t
- 打印版本信息 -v -V

```
nginx             //启动
nginx -s reload   // 重启
nginx -s quit    //优雅停止
nginx -s stop    //直接停止
nginx -s reopen  //重新开始记录日志
```

#### 热部署

当你需要升级nginx或者增加模块的时候可以使用热部署的方式平滑过渡，不需要重启

查看现在nginx进程的id
```bash
ps -ef|grep nginx
```

对现在的nginx进程发送`USR2`信号进行热部署

```
kill -USR2 id
```

现在可以看到新的nginx进程和老的nginx进程都存在了，这个时候我们需要让老的nginx进程平滑退出

```
kill -WINCH id
```

接下来就可以看到老的nginx的worker进程全部平滑退出了，但是老的master进程还在，这是为了防止新的有问题，还可以退回老的

如果接下来新的有问题，那么我们退回老的，发送`HUP`信号，而不是直接reload

```
kill -HUP id
```

接下来可以看到老的worker进程都起来了

如果热部署后没有问题的话，那么我们直接删除老的master进程就可以了

```
kill -QUIT id
```

#### 日志切割

如果我们觉得日志太大可以进行日志切割，一种是发送`reopen`信号，还有一种是发送`USR1`信号

```
sudo nginx -s reopen
```

或者

```
kill -USR1 id
```

但是注意切割之前要把之前的日志备份一下，不然就会被删除啦


## 反向代理

反向代理配置在`http`代码块中

首先通过`upstream`配置反向代理的服务器

test 是一个名称，可以随便取

```nginx
upstream test{
    server 127.0.0.1:8080     # 上游服务器的地址，可以写多个
}
```

通过 `proxy_pass` 进行反向代理

```
server{
    listen 80;
    location / {
        proxy_pass http://test;   # 这里填写上面的upstream名字
    }
}
```

这个时候我们访问80端口实际上访问的是8080端口


#### 反向代理的缓存

反向代理之后还可以在反向代理服务器做缓存

在`nginx.conf`文件的`http`代码块中设置缓存

```
# 设置缓存路径 /tmp/nginxCache 缓存名称 my_cache
proxy_cache_path /tmp/nginxCache levels=1:2 keys_zone=my_cache:10m max_size=10g inactive=60m use_temp_path=off;
```

接下来在我们的反向代理`server/location`代码块里面使用这个缓存

```
location / {
    proxy_cache my_cache;   # 使用上面定义的缓存
    proxy_cache_key $host$uri$is_args$args;  # 缓存的key值，根据host uri和参数来做key
    proxy_cache_valid 200 304 302 1d;   # 缓存200 304 302状态 缓存时间1d = 1天
    proxy_pass http://test;   # 这里填写上面的upstream名字
}
```


## goaccess 日志可视化

goaccess 是一个c语言写的日志可视化工具

直接编译安装

```
wget http://tar.goaccess.io/goaccess-1.2.tar.gz
tar -xzvf goaccess-1.2.tar.gz
cd goaccess-1.2/
./configure --enable-utf8
make
make install
```

然后就可以启动了，直接启动就可以

指定日志文件路径 `/usr/local/src/nginx/logs/access.log` -o 输出到某个html文件中 --real-time-html 代表实时刷新, --log-format 代表日志格式 ，--daemonize 代表守护进程后台执行 仅在 开启了 --real-time-html 实时刷新有效

```
goaccess /usr/local/src/nginx/logs/access.log -o ~/wwwroot/goaccess.html --real-time-html --log-format=COMBINED --daemonize
```

这里要注意，如果开启了  --real-time-html 实时刷新，那么会启动一个 socket 进程监听 7890 端口，所以 7890 端口要开放才行

可以配置 nginx 文件访问这个输出的 html 文件了，就可以看到一个可视化界面了


## ssl

ssl(Secure Sosckets Layer) 
tls(Transport Layer Security)

- 1995年-ssl3.0
- 1999 - tls1.0
- 2006 - tls1.1
- 2008 - tls1.2
- 2018 - tls1.3

ssl/tls 运行在 `ISO7层模型的表示层中`，而http协议运行在表示层的上层`应用层`。在http的数据传输到`表示层`以后，ssl/tls就会对数据进行加密

- 握手
- 交换秘钥
- 告警
- 对称加密的应用数据

tls安全套件

> TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256

- tls代表tls加密
- ECDHE 代表秘钥交换算法
- RSA 代表身份验证算法
- AES 代表对称加密算法
- 128 代表强度
- GCM 代表分组模式
- SHA256 代表签名 hash 算法


