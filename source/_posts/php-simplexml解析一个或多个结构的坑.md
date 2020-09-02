---
title: php-simplexml解析一个或多个结构的坑
date: 2020-08-25 10:12:47
tags: ['php','xml']
category: php
article: php-simplexml解析一个或多个结构的坑
---

# php-simplexml解析一个或多个结构的坑

php解析xml还是挺方便的，不管是正常的xml，还是加了一个命名空间或者前缀的xml。都可以通过`simplexml_load_string`函数来解析成数组或者对象。

## simplexml_load_string

来看一下使用方法。

```php
$xml = "<reports><report><id>1</id><name>张三</name></report></reports>"

$arr = (array)simplexml_load_string($xml);
```

是不是很简单，如果你的带啦前缀或者命名空间也可以使用。下面带了s前缀


```php
$xml = "<s:reports><s:report><s:id>1</id><s:name>张三</name></report></reports>"

$arr = (array)simplexml_load_string($xml,'SimpleXMLElement',0,'s',true);
```

但是如果带了多个前缀，这个函数就无能为力了，可以使用别的方法解析。

## bug

不过这个函数解析一个和多个结果是不一样的，这里解析出来一定要做判断！！！

下面有两个report，解析出的结果是一个数组。

```php
$xml = "<reports><report><id>1</id><name>张三</name></report><report><id>1</id><name>张三</name></report></reports>"

$arr = (array)simplexml_load_string($xml);
```

而上面只有一个report的时候，解析出来就是一个对象！！！