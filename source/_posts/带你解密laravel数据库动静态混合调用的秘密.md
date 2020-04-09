---
title: 带你解密laravel数据库动静态混合调用的秘密
date: 2020-04-09 15:28:49
tags: ['php','laravel']
category: php
article: 带你解密laravel数据库动静态混合调用的秘密
---

# laravel orm

laravel 使用 orm 调用数据库查询的时候很方便，我们只需要配置完数据库连接后创建model。

比如我们查询用户，我们首先要有一个`user model`。

可以使用 `artisan`创建

> php artisan make:model userModel

如果我们需要查询一个用户信息，只需要这样

```php
userModel::where('id', 1)->first();
(new userModel)->where('id', 1)->first();
```

上面的两种方法都可以实现查询用户id为1的用户信息。那他们有什么区别呢？第一种方法更为优雅。

那这个是怎么实现的呢，一般我们要么定义一个静态方法用来静态调用，要么定义一个对象方法需要使用对象调用。

其实很简单，他只是用到了`php`的`魔术方法`。

来看一下下面两个魔术方法：

- __call()
- __callStatic()

看一下php文档中的介绍

### __call()

在对象中调用一个不可访问方法时，__call() 会被调用。

> public __call ( string $name , array $arguments ) : mixed

### __callStatic()

在静态上下文中调用一个不可访问方法时，__callStatic() 会被调用。

> public static __callStatic ( string $name , array $arguments ) : mixed

$name 参数是要调用的方法名称。$arguments 参数是一个枚举数组，包含着要传递给方法 $name 的参数。

我们要让一个对象方法可以静态调用就要通过`__callStatic()`魔术方法了。

```php

public static function __callStatic( string $name , array $arguments) {
    return (new static)->$name(...$arguments);
}

```

这样的话当我们调用`userModel::where()`的时候实际上它内部会创建一个对象然后再对象调用`where`。

但是这样会产生一个问题，如果当我们调用的方法不存在的时候怎么办呢。

可以通过`try catch`来捕获错误进行错误处理。laravel内部就是这么实现的。在laravel的model文件下，是这么运用`__call()`的。

```php

/**
     * Handle dynamic method calls into the model.
     *
     * @param  string  $method
     * @param  array  $parameters
     * @return mixed
     */
    public function __call($method, $parameters)
    {
        if (in_array($method, ['increment', 'decrement'])) {
            return $this->$method(...$parameters);
        }

        return $this->forwardCallTo($this->newQuery(), $method, $parameters);
    }

```

他里面调用了`forwardCallTo`方法，我们来看一下这个方法。

这个方法存在`ForwardsCalls`这个`trait`中。

```php

/**
     * Forward a method call to the given object.
     *
     * @param  mixed  $object
     * @param  string  $method
     * @param  array  $parameters
     * @return mixed
     *
     * @throws \BadMethodCallException
     */
    protected function forwardCallTo($object, $method, $parameters)
    {
        try {
            return $object->{$method}(...$parameters);
        } catch (Error | BadMethodCallException $e) {
            $pattern = '~^Call to undefined method (?P<class>[^:]+)::(?P<method>[^\(]+)\(\)$~';

            if (!preg_match($pattern, $e->getMessage(), $matches)) {
                throw $e;
            }

            if ($matches['class'] != get_class($object) ||
                $matches['method'] != $method) {
                throw $e;
            }

            static::throwBadMethodCallException($method);
        }
    }

```

他在调用的时候通过try catch来捕获错误。

还有另外一种方法同样可以判断这个类有没有这个方法，那就是先通过`get_class_methods($className)`这个函数获取到类的所有方法，然后我们就可以自己判断了。

```php

    public function __call($method, $parameters)
    {
        $classFuns = get_class_methods($this);
        if (!in_array($method, $classFuns)) {
            return "没有这个方法"; //返回错误
        }

        return $this->$method(...$parameters);
    }
```

这样也不失为一种方法啊，大家还有其他好方法的话欢迎交流。
