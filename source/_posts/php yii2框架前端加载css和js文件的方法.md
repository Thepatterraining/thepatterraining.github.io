---
title: php yii2框架前端加载css和js文件的方法
date: 2020-08-21 10:12:47
tags: ['php','js','yii2']
category: php
article: php yii2框架前端加载css和js文件的方法
---

# php yii2框架前端加载css和js文件的方法

这两天有一个以前的项目是用`yii2`框架写的，前后端没有做分离，现在需要用vue接手后续的前端开发。

把vue的项目放到yii2里面，这时候遇到一个加载静态资源的问题，原来html的引用方式不管用了。

后来看到yii2官方文档里面，需要改一下引用方式。

改成下面这样就可以了。

```php
$this->registerCssFile("@web/static_vue/css/index.css")
$this->registerJsFile("@web/static_vue/js/index.js")
```

所有都使用这两个php代码进行引入，引入后就可以了。



