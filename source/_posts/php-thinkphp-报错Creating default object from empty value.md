---
title: php-thinkphp-报错Creating default object from empty value
date: 2020-08-25 10:12:47
tags: ['php','thinkphp']
category: php
article: php-thinkphp-报错Creating default object from empty value
---

# php-thinkphp-报错Creating default object from empty value

报错第一步，打印，打印日志，在你用到对象的地方，把对象都打印出来看看，你以为他是个对象，但他。。。不一定是个对象！！！

如果你确定你从数据库查询出来的没有错，是个对象，那么。。请看一下你别的对象是不是一个对象！！！

要相信报错，一定是对象错了，但你不确定，所以，请打印日志，如果你这里没问题，别人那里有问题，那么可能是参数的问题。打印日志在别人那就能看到你想的对象可能在他那里不是一个对象！！！