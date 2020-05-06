---
title: 解决composer-npm等包管理工具下载失败的问题
date: 2020-05-03 16:55:35
tags: ['php','composer', 'javascript', 'npm']
category: 工具
article: 解决composer-npm等包管理工具下载失败的问题
---

# 解决composer-npm等包管理工具下载失败的问题

使用composer npm进行下载依赖包的时候经常需要翻墙。不过国内已经有了很多国内源来加速下载，比如阿里云的国内镜像等。

不过有时候国内镜像并不能解决问题。

还有的时候就算你挂了全局翻墙也不行。

这是因为你使用命令行的时候，是用不到翻墙工具的。

如果你有翻墙工具，那么可以在命令行上面设置一下，使用代理就可以了。

在gitbash上面的设置。

> export http_proxy=http://127.0.0.1:41091  //这个端口是你本地代理工具提供的端口
> export https_proxy=http://127.0.0.1:41091 //这个端口是你本地代理工具提供的端口

在cmd上的设置

> set http_proxy=http://127.0.0.1:41091  //这个端口是你本地代理工具提供的端口
> set https_proxy=http://127.0.0.1:41091 //这个端口是你本地代理工具提供的端口