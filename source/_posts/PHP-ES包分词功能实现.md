---
title: PHP-ES包分词功能实现
date: 2022-01-19
tags: ['ES','php']
category: ES
article: PHP-ES包分词功能实现
---

# 安装

直接使用 composer 安装 ES 包就可以了，这里使用官方的 `elasticsearch/elasticsearch` 这个包。

```
composer require elasticsearch/elasticsearch
```

安装好以后，创建一个客户端。hosts如果是多个节点的集群，那么可以配置一个二维数组。

```php
$hosts = [
            'host' => '127.0.0.1',
            'port' => '9200',
            'scheme' => 'http',
            'user' => '',
            'pass' => ''
        ];
$client = ClientBuilder::create()  //创建客户端
                        ->setHosts($hosts) //hosts连接地址
                        ->build();
```

如果想跳过ssl证书校验，可以添加一些curl的参数放进客户端

```php
$curlParams = [//不校验ssl
                CURLOPT_SSL_VERIFYPEER => 0,
                CURLOPT_SSL_VERIFYHOST => 0,
            ];
$client = ClientBuilder::create()
            ->setConnectionParams(['client' => ['curl' => $curlParams]]) //设置curl参数
            ->setHosts($hosts)
            ->build();
```

### 分词

简单的增删改查在 elasticsearch 的文档中有介绍了，就不说了，可以看(https://www.elastic.co/guide/cn/elasticsearch/php/current/_quickstart.html)[文档]。

分词的话隐藏的比较深，文档中没有介绍，他放在了`indices`这个 namespace 下面。如果看源码，可以在`Endpoints/Indices` 目录下面发现 `Analyze.php` 文件，当然了，除了分词，这里面还有其他功能，可以自己看。

这个文件也很简单啊，只有几个函数，就是设置请求的API地址，参数这些。

使用起来是这样的，我们用上面创建好的客户端。

```php
$parmas['index'] = 'test' //这个是分词的index，也可以不加，加了请求的API就是 $index/_analyze
//请求体 这个就和你直接写DSL没区别了，参数啥的都一样，可以在 kibana里面试试参数
$parmas['body'] = [
  'text' => 'php开发', //要分词的文字
  'analyzer' => 'ik_smart', //分词器，可以不写
]; 
$client->indices()->analyze($params);
```

### 扩展

indices函数返回的就是 indices 文件夹的 namespace，对应文件在`namespace/IndicesNamespace.php`,然后后面的函数就相当于文件名，她会拼接在 indices 后面，像我们上面请求的文件就是`indices/Analyze.php`。具体的拼接就是在 `namespace/IndicesNamespace.php` 这个里面做的，有一个函数如下

```php
/**
  * $params['index'] = (string) The name of the index to scope the operation
  * $params['body']  = (array) Define analyzer/tokenizer parameters and the text on which the analysis should be performed
  *
  * @param array $params Associative array of parameters
  * @return array
  * @see https://www.elastic.co/guide/en/elasticsearch/reference/master/indices-analyze.html
  */
public function analyze(array $params = [])
{
    //获取$params['index'], $params['body']
    $index = $this->extractArgument($params, 'index');
    $body = $this->extractArgument($params, 'body');

    //设置endpoints处理类
    $endpointBuilder = $this->endpoints;
    //$endpoint 就相当于 indices/Analyze.php 这个文件了
    $endpoint = $endpointBuilder('Indices\Analyze');
    //设置参数，index, 请求体
    $endpoint->setParams($params);
    $endpoint->setIndex($index);
    $endpoint->setBody($body);
    //发起请求
    return $this->performRequest($endpoint);
}
```

$this->endpoints 是一个函数，外面传进来的，函数内容如下

```php
$this->endpoint = function ($class) use ($serializer) {
  //拼接处理类
  $fullPath = '\\Elasticsearch\\Endpoints\\' . $class;
  //反射获取
  $reflection = new ReflectionClass($fullPath);
  $constructor = $reflection->getConstructor();
  //执行
  if ($constructor && $constructor->getParameters()) {
      return new $fullPath($serializer);
  } else {
      return new $fullPath();
  }
};
```
