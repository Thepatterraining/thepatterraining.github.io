---
title: JAVA-spring-IOC
date: 2022-11-03
tags: ['java','spring','IOC']
category: java
article: JAVA-spring-IOC
---

# spring

## IOC控制反转

这里先说一下`IOC`，再说`IOC`在`spring`框架中的使用。

### IOC的概念

`IOC`这个缩写有很多意思，比如
- 智慧城市智能运行中心(IOC)
- 奥林匹克运动的领导机构

但是呢，我们这里说的是`面向对象编程中的一种设计原则`。他的全称是`Inversion Of Control`即控制反转。这里有两个单词`控制`和`反转`。这两个单词单独拿出来会发现，都缺少主语。比如
- 谁控制了谁?
- 什么东西发生反转了呢?

#### 控制

这里说一下第一个问题，谁控制了谁呢？看下面的代码

```java

public class main{

    public static void main(String[] args) {
        Person person = new Person();
        person.setName("tony");
        System.out.println(person.getName());
    }
}

class Person{
    private String name;

    public void setName(String name) {
        this.name = name;
    }

    public String getName() {
        return this.name;
    }
}

```

很显然，在这里，Person类的person对象的一切都控制在main函数里面。main函数创建它，使用它，销毁它。所以在当前上下文中，main控制了person。

#### 反转

上面的写法main函数控制了person对象，这是一种紧耦合的关系，如果person发生了改变，我们就需要改变main。`反转`是控制权的反转。现在person的控制权在main这里，我们将它反转一下。不在用main控制它。那么我们加一个简单工厂看一下呢。

```java

class Factory{
    public static Person createPerson() {
        return new Person();
    }
}

public static void main(String[] args) {
    Person person = Factory.createPerson();
    person.setName("tony");
    System.out.println(person.getName());
}

```

这里可以看到person的控制权转交给了`Factory`工厂，而main只有使用权了。当然了作为示例代码，这里的控制只做了创建。

现在，由main控制person改为了Factory控制person。如果person发生了改变，我们只需要改变Factory，而不需要动业务逻辑。

当然了，这种程度的解耦依然不够，因为main还是和Factory有耦合关系，他还控制了Factory。我们可以扩大这个简单工厂，扩大后的简单工厂就不再是工厂了，而叫做`容器`。我们把所有的控制权都交给容器，让容器控制所有的类，对象。而在使用的时候我们去告诉容器我们要使用哪个对象，让容器给我们提供就可以了。


#### DI

`依赖注入 DI`全称`Dependency Injection`就可以实现我们告诉容器我们需要的对象，然后容器把对象注入给我们的功能。

具体的依赖注入实现方式每个语言，每个框架可能都不一样，这里以spring为例

```java
public class main{

    @Autowired
    private Person person;
    
    public static void main(String[] args) {
        person.setName("tony");
        System.out.println(person.getName());
    }
}

```

可以看到，我们通过`@Autowired`注解告诉容器，我们需要一个Person类型的对象，然后让容器把这个对象注入到我们的person属性中。这里仅做示例使用，具体情况请以实际开发中为准，实际开发中应使用接口类型。


容器中的实现方式就类似刚才工厂中的，假设你需要Person类型的对象，就`new Person`返回，当然了，要更加复杂，比如可以根据名称来反射创建对象，可以更好的管理对象的生命周期，可以实现单例对象等等。


## spring中的IOC



