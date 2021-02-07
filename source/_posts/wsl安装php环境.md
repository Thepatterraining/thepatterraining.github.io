---
title: wsl安装php环境
date: 2020-08-21 10:12:47
tags: ['php']
category: php
article: wsl安装php环境
---

# wsl

wsl是可以在windows里面运行linux的一个软件。是微软官方发行的。

## 安装php

从php官网下载php tar包。

```
 sudo wget https://www.php.net/distributions/php-7.4.12.tar.gz
```

然后解压

```
sudo tar -zxvf php-7.4.12.tar.gz
```

接下来需要安装一些扩展来支持php。

```
sudo apt-get install gcc make pkg-config libxml2-dev libssl-dev libsqlite3-dev libcurl4-openssl-dev libonig-dev zlib1g-dev libffi-dev libpng-dev libzip-dev
```

不安装上面的扩展会导致接下来报错。

切换目录

```
cd ./php-7.4.12
```

执行configure,注意这里`prefix`一定要是/usr/local/php7，要不然找不到配置文件php.ini。这里有这个坑。

```
sudo ./configure --enable-fpm --with-mysql --with-pear --with-zip --enable-sockets --enable-soap --with-pdo-mysql  --enable-gd --enable-ftp --with-ffi  --with-zlib  --with-curl --with-openssl --enable-mbstring --prefix=/usr/local/php7 --with-config-file-path=/usr/local/php7 --with-external-gd --with-webp  --with-jpeg  --with-xpm  --with-freetype  --enable-bcmath
```

执行完上面一步如果没有错误就可以了。

接下来执行make

```
sudo make && sudo make install
```

建立软连接或者环境变量。我们要配置全局的环境变量有两种方式。
- 在环境变量目录里面增加软连接
- 把php目录增加到环境变量里面

我采用的是软连接的方式。

```
sudo ln -s /usr/local/php7/bin/php /usr/local/bin/php
sudo ln -s /usr/local/php7/bin/phpize /usr/local/bin/phpize
sudo ln -s /usr/local/php7/sbin/php-fpm /usr/local/bin/php-fpm
```

接下来就可以全局使用php命令了

### php-fpm启动，重启方法

启动

```
sudo php-fpm
```

重启 先找到进程 然后发送`USR2`信号

```
ps -aux | grep php

sudo kill -USR2 进程id
```


## 安装nginx

访问nginx的[官网](https://nginx.org/en/download.html)进行下载。

复制下载地址。比如我下载的1.19.5,直接下载

```
sudo wget https://nginx.org/download/nginx-1.19.5.tar.gz
```

然后解压

```
sudo tar -zxvf nginx-1.19.5.tar.gz
```

接下来需要安装一些扩展来支持php。

```
sudo apt-get install libpcre3 libpcre3-dev
```

切换目录

```
cd ./nginx-1.19.5
```

执行安装

```
sudo ./configure --prefix=/usr/local/src/nginx
sudo make && sudo make install
```

好了，安装完成。

同意，建立软连接。

```
sudo ln -s /usr/local/src/nginx/sbin/nginx /usr/local/bin/nginx
```

接下来启动nginx看看效果

```
sudo nginx
```

访问localhost就可以看到效果了。


## 配置网站

接下来配置一下网站。

修改nginx.conf

```
sudo vim /usr/local/src/nginx/conf/nginx.conf
```

在http块里面加上这句话，引入其他的配置文件

```
include     conf.d/*.conf;
```

然后我们创建这个目录

```
sudo mkdir /usr/local/src/nginx/conf/conf.d
```

配置我们的网站文件

```
sudo vim /usr/local/src/nginx/conf/conf.d/test.com.conf
```

复制下面内容

```
server {
        listen       80;
        server_name  test.com;
        root   "/home/wwwroot/test/public/";
        location / {
            index index.php index.html error/index.html;
            autoindex  off;
            if (!-e $request_filename) {
                rewrite ^(.*)$ /index.php?s=/$1 last;
                break;
            }
        }
        location ~ \.php(.*)$ {
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_split_path_info  ^((?U).+\.php)(/?.+)$;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            fastcgi_param  PATH_INFO  $fastcgi_path_info;
            fastcgi_param  PATH_TRANSLATED  $document_root$fastcgi_path_info;
            include        fastcgi_params;
        }
}
```

在修改hosts文件就好了。

## redis扩展

如果要装redis扩展，那么手动下载

```
http://pecl.php.net/package/redis
```

找到tar包下载

```
sudo wget http://pecl.php.net/get/redis-5.3.2.tgz
```

解压缩
```
sudo tar -zxvf redis-5.3.2.tgz
```

进去执行phpize

```
cd ./redis-5.3.2
sudo phpize
```

然后编译

```
sudo ./configure --with-php-config=/usr/local/php7/bin/php-config
sudo make && sudo make install
```

接下来会出现下面的目录

```
/usr/local/php7/lib/php/extensions/no-debug-non-zts-20190902
```

修改我们的php.ini

```
sudo vim /usr/local/php7/lib/php.ini
```

修改下面这个
```
extension_dir=/usr/local/php7/lib/php/extensions/no-debug-non-zts-20190902
```

增加redis扩展

```
extension="redis.so"
```

重启nginx和php-fpm就好了。

