---
title: OpenResty的安装
date: 2021-06-07
tags: ['OpenResty','wsl']
category: OpenResty
article: OpenResty的安装
---

# OpenResty的安装

wsl 使用的是ubuntu 20的版本，所以可以直接使用`apt`进行安装，OpenResty官网的安装也是推荐使用的apt进行安装。

添加openresty的公钥

```
sudo apt-get -y install --no-install-recommends wget gnupg ca-certificates
```

导入 GPG 秘钥

```
wget -O - https://openresty.org/package/pubkey.gpg | sudo apt-key add -
```

添加 Openresty 的源

```
echo "deb http://openresty.org/package/ubuntu $(lsb_release -sc) main" > openresty.list
sudo cp openresty.list /etc/apt/sources.list.d/
```

更新apt

```
sudo apt-get update
```

安装openresty

```
sudo apt-get -y install --no-install-recommends openresty
```

检查版本确认是否安装成功

```
openresty -V
```

启动 openresty

```
sudo service openresty start
```

访问 localhost 查看

```
curl localhost
```

安装 resty 命令

```
sudo apt-get -y install openresty-resty
```

执行 hello world看看是否安装成功

```
resty -e 'print("Hello Resty")'
```


安装 resty-doc

```
sudo apt-get -y install openresty-restydoc
```
