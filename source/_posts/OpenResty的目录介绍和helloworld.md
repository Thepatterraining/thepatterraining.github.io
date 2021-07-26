---
title: OpenResty的目录介绍和helloworld
date: 2021-06-07
tags: ['OpenResty','wsl']
category: OpenResty
article: OpenResty的目录介绍和helloworld
---

# OpenResty的目录介绍和helloworld

## 安装目录介绍

上文提到是 ubuntu apt 安装的 openresty

可以查看安装目录

```
openresty -V
```

安装目录是`/usr/local/openresty`

目录：
- bin 这个目录里面是可执行文件，比如openresty,resty,restydoc这些程序
- pod 这个目录存放的是openresty的文档
- nginx 存放的就是nginx得相关文件
- luajit 存放的luajit的相关文件
- lualib 这个目录放的是lua库，主要分为 ngx 和 resty 两个子目录
    - ngx 存放 lua-resty-core 这个官方项目中的lua代码，里面都是基于FFI重新实现的 openresty API
    - resty 存放各种 lua-resty-* 项目包含的lua代码

    