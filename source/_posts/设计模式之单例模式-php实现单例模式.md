---
title: 设计模式之单例模式--php实现单例模式
date: 2020-05-02 14:47:41
tags: ['设计原则','设计模式']
category: 设计模式
article: 设计模式之单例模式--php实现单例模式
---

# 设计模式之单例模式--php实现单例模式

`单例模式`是最常被提起的一个设计模式，他的意思是

> 一个类只有一个实例，所有人共用这同一个实例。不让外部创建类的实例，并且提供一个外部的全局访问点。

## 为什么要使用单例模式

单例模式有什么好处呢？

单例模式是为了全局唯一，如果你需要一个全局唯一的类那么就需要单例模式了。而且单例模式还可以节省资源，因为这个类只有一个对象。

比如你需要一个全局唯一的id生成器，如果创建了多个实例，那么生成的id可能就不是全局唯一了。

比如你需要一个全局配置信息，需要全局缓存信息，全局日志文件写入等等。

## 普通类

下面是一个普通的类的实现和使用。

```php
class Common {
    public $name; //姓名
    public $age;  //年龄

    function __construct($name, $age)
    {
        $this->name = $name;
        $this->age = $age;
    }

    public function print() {
        dump('姓名：'.$this->name);
        dump('年龄：'.$this->age);
    }
}

$obj1 = new Common('张三', '18');
dump($obj1);

$obj2 = new Common('张三', '20');
dump($obj1);

```
可以看到普通类可以`new`多个对象，并且每个对象都不一样，拥有自己的属性。

## 单例类

首先，单例类是一种特殊的类，单例类只允许`new`一次。怎么实现只能`new`一次呢，从上面的代码中可以看到，`new`这个操作是在我们的业务中进行的，这是个无法掌控的操作。

我们要让`new`操作在类中实现，由这个单例类来决定，来操作对象的实例化。而不是在业务中进行。

只有自己才能决定自己`new`不`new`，自己决定自己的人生，而不是交给别人，使得无法掌控。

从这个分析中可以知道，单例类要完成下面的操作：

- 自己创建实例化，供外部全局访问
- 不允许外部实例化

第一点简单，实现一个方法`new`自己，然后存起来返回就可以了。

第二点在php中怎么实现呢，不允许外部实例化可以通过将`__construct`方法设置为`private`权限。因为这个方法外部访问不到，所以外部就无法实例化了。

简单实现一下单例类。

```php
class Singleton {
    public $name; //姓名
    public $age;  //年龄
    private static $instance;

    private function __construct($name, $age)
    {
        $this->name = $name;
        $this->age = $age;
    }

    public static function getInstance($name, $age) {
        if (empty(self::$instance)) {
            self::$instance = new self($name, $age);
        }
        return self::$instance;
    }
    
    public function print() {
        dump('姓名：'.$this->name);
        dump('年龄：'.$this->age);
    }
}

$obj1 = Singleton::getInstance('张三', '18');
dump($obj1);
$obj2 = Singleton::getInstance('张三', '20');
dump($obj2);
```

这是把上面的普通类改成了单例类，现在这个单例类还有些问题。

- 参数上面第二个参数没有生效

单例类一般不会出现这种需要各种参数的，需要参数也建议通过配置文件来搞定参数，而不是传参。**如果有传参需求，需要的是不同的参数是不同的对象，同一个参数是单例对象，那么需要维护一个单例对象的数组**。为什么不提供修改参数的方法呢，因为提供这个方法太危险，全局使用同一个对象，如果某个地方不慎更改了属性，那么会导致其他地方出现未知bug。

- 并没有完全禁止外部实例化，php还有魔术方法__clone。

把__clone方法设置成`private`

根据上面的我们在修改一下这个单例类。

```php

class Singleton {
    protected $name; //姓名
    protected $age;  //年龄

    //用来存放全局唯一对象
    private static $instance;

    /**
     * 私有化构造函数使得外部无法new对象
     * 构造函数去掉参数，参数从配置文件读取
     */
    private function __construct()
    {
        $this->name = config('design.singleton.name');
        $this->age = config('design.singleton.age');
    }

    /**
     * 私有化克隆函数使得外部无法克隆
     */
    private function __clone() {

    }

    /**
     * 获取全局唯一对象的时候不需要传参
     */
    public static function getInstance() {
        if (empty(self::$instance)) {
            self::$instance = new self();
        }
        return self::$instance;
    }
    
    public function print() {
        dump('姓名：'.$this->name);
        dump('年龄：'.$this->age);
    }
}

$obj1 = Singleton::getInstance();
dump($obj1);
$obj2 = Singleton::getInstance();
dump($obj2);
```

到现在，我们就完成了一个基于php的单例类，这个单例类支持延迟加载。如果不需要延迟加载，那么实现更简单。看下面的`饿汉式单例类`

```php

class Singleton {
    protected $name; //姓名
    protected $age;  //年龄

    //用来存放全局唯一对象 饿汉式单例直接在这里new
    private static $instance = new self();

    /**
     * 私有化构造函数使得外部无法new对象
     * 构造函数去掉参数，参数从配置文件读取
     */
    private function __construct()
    {
        $this->name = config('design.singleton.name');
        $this->age = config('design.singleton.age');
    }

    /**
     * 私有化克隆函数使得外部无法克隆
     */
    private function __clone() {

    }

    /**
     * 获取全局唯一对象的时候不需要传参
     * 这里只需要直接返回
     */
    public static function getInstance() {
        return self::$instance;
    }
    
    public function print() {
        dump('姓名：'.$this->name);
        dump('年龄：'.$this->age);
    }
}

```

我们支持延迟加载的其实还有些问题，那就是缺少锁，如果很多请求同时进来，那么后面的会覆盖前面的单例对象。如果再加上锁的话实现是下面这样。


```php

class Singleton {
    protected $name; //姓名
    protected $age;  //年龄

    //用来存放全局唯一对象
    private static $instance;

    /**
     * 私有化构造函数使得外部无法new对象
     * 构造函数去掉参数，参数从配置文件读取
     */
    private function __construct()
    {
        $this->name = config('design.singleton.name');
        $this->age = config('design.singleton.age');
    }

    /**
     * 私有化克隆函数使得外部无法克隆
     */
    private function __clone() {

    }

    /**
     * 获取全局唯一对象的时候不需要传参
     */
    public static function getInstance() {
        //先加锁
        $lock = new lock;
        $lock->lock();
        if (empty(self::$instance)) {
            self::$instance = new self();
        }
        //释放锁
        $lock->unlock();
        return self::$instance;
    }
    
    public function print() {
        dump('姓名：'.$this->name);
        dump('年龄：'.$this->age);
    }
}
```

这样的话因为锁的原因会造成性能比较低，所以又有了下面的方式，叫做`双重检测`。也就是拿锁之前先判断一下有没有实例化，如果有了就不需要锁了。为什么不直接在第一个判断里面加呢，因为锁加晚了没有意义了。

```php

class Singleton {
    protected $name; //姓名
    protected $age;  //年龄

    //用来存放全局唯一对象
    private static $instance;

    /**
     * 私有化构造函数使得外部无法new对象
     * 构造函数去掉参数，参数从配置文件读取
     */
    private function __construct()
    {
        $this->name = config('design.singleton.name');
        $this->age = config('design.singleton.age');
    }

    /**
     * 私有化克隆函数使得外部无法克隆
     */
    private function __clone() {

    }

    /**
     * 获取全局唯一对象的时候不需要传参
     */
    public static function getInstance() {
        //先判断
        if (empty(self::$instance)) {
            //先加锁
            $lock = new lock;
            $lock->lock();
            if (empty(self::$instance)) {
                self::$instance = new self();
            }
            //释放锁
            $lock->unlock();
        }
        
        return self::$instance;
    }
    
    public function print() {
        dump('姓名：'.$this->name);
        dump('年龄：'.$this->age);
    }
}
```

这就是整个单例模式。四种实现：
- 饿汉式 在类加载的时候实例化
- 懒汉式 支持延迟加载
- 带锁懒汉式 延迟加载的时候加锁
- 双重检测 两次判断

代码放在了我的github上面。

- [设计模式](https://github.com/Thepatterraining/design-pattern)