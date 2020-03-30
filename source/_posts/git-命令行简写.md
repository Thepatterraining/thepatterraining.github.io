---
title: git 命令行简写
date: 2018-03-23
tags: [git, config]
category: git
article: "git 命令行简写"
---

常用的 `git` 命令不多，反正我就是那几个
``` git
    git status
    git branch
    git checkout
    git commit
    git diff
```

虽然看起来也不长，但是我还是喜欢简写，哈哈，说一下怎么配置

<!--more-->


>vim ~/.gitconfig    打开配置文件

输入如下

    [alias]
            br = branch
            st = status
            co = checkout
            cm = commit
            df = diff
            dt = difftool
    [diff]
            tool = vimdiff

    [user]
            name = xxx
            email = xxx

`alias` 就是简写啦，这样的话，使用就方便多了，`diff` 和 `user` 大家可以不用管，哈哈

## 推荐个小工具

给大家安利个小工具 `gitbash` , 用起来很方便，他会在你的命令行里面直接显示出你的分支。下面是 github 的地址，操作简单,大家照着弄一下就好了

>https://github.com/mocheng/gitbash