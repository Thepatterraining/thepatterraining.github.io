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

`Spring`中的IOC实现也是通过容器，但是怎么把类注入到容器中呢，也就是怎么告诉容器，你需要实例化哪些类呢？有两种方式，一种是`XML配置方式`，一种是`注解方式`。

### XML配置方式

先创建一个XML文件，比如:
> touch bean.xml

接下来编辑它。
> vim bean.xml

#### 无参数构造方式

重点在这个配置，通过`bean`这个标签来告诉容器我要把哪些类注册到容器中，其中id就是注册后的唯一标识，我们获取的时候也可以通过指定id来从容器中获取对象。而`class`是告诉容器，我们具体要注入的类的路径。但是这时候没有指定参数，也就是说类似于`Person person = new Person()`这样的注册，需要有无参构造器。

>  <bean id="person" class="com.test.java.Person"></bean>

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

   <bean id="person" class="com.test.java.Person"></bean>

</beans>
```

使用的时候需要借助于`Spring`提供的容器获取,可以看一下效果是一样的。

```java
public class main{
    
    public static void main(String[] args) {
        //这里先加载我们的XML配置文件
        BeanFactory ac = new ClassPathXmlApplicationContext("bean.xml");
        //通过容器获取一个对象,第一个参数也就是配置的id,第二个就是类
        Person person = ac.getBean("person", Person.class);
        person.setName("tony");
        System.out.println(person.getName());
    }
}

```

#### 构造器构造参数方式

同样在上面的XML文件中进行修改，但这次我们需要加上参数，并且告诉容器是通过构造器来构造参数的，而不是`set`的方式。

我们只需要在原来的`bean`标签中，加入我们要传给构造器的参数就可以了。

使用`constructor-arg`标签，`name`就是参数名，`value`是对应的值。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="person" class="com.test.java.Person">
       <constructor-arg name="name" value="tony"></constructor-arg>
    </bean>

</beans>
```

我们还需要改造一下原来的`Person`类，增加一个构造函数.

```java
class Person{
    private String name;

    public Person(String name) {
        this.name = name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getName() {
        return this.name;
    }
}

```

看一下现在的main函数

```java
public class main{
    
    public static void main(String[] args) {
        //这里先加载我们的XML配置文件
        BeanFactory ac = new ClassPathXmlApplicationContext("bean.xml");
        //通过容器获取一个对象,第一个参数也就是配置的id,第二个就是类
        Person person = ac.getBean("person", Person.class);
        // 这里不需要setName了，因为通过构造器注入参数了
        // person.setName("tony");
        System.out.println(person.getName());
    }
}

```

但是如果我们需要构造一个其他类的对象作为参数该怎么配置呢，毕竟我们总不能在`value`上面写`new Class()`吧哈哈。但是我们知道一个`bean`就是一个对象，那我们可以传一个`bean`进来就可以了。

来看一下配置方式.不再使用`value`了，因为`value`只能传基本类型这些，而其他的对象需要使用`ref`来传参。`ref`的值就是其他`bean`的`id`。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="avatar" class="com.test.java.Avatar">
         <constructor-arg name="url" value="http://baidu.com"></constructor-arg>
    </bean>

    <bean id="person" class="com.test.java.Person">
       <constructor-arg name="name" value="tony"></constructor-arg>
       <constructor-arg name="avatar" ref="avatar"></constructor-arg>
    </bean>

</beans>
```

同样需要改造一下构造函数，增加一个`avatar`参数。

```java
class Avatar{
    private String url;
    public Avatar(String url) {
        this.url = url;
    }
}

class Person{
    private String name;

    private Avatar avatar;

    public Person(String name, Avatar avatar) {
        this.name = name;
        this.avatar = avatar;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getName() {
        return this.name;
    }

    public String getAvatar() {
        return this.avatar.url;
    }
}

```

执行一下`main`函数。

```java
public class main{
    
    public static void main(String[] args) {
        //这里先加载我们的XML配置文件
        BeanFactory ac = new ClassPathXmlApplicationContext("bean.xml");
        //通过容器获取一个对象,第一个参数也就是配置的id,第二个就是类
        Person person = ac.getBean("person", Person.class);
        // 这里不需要setName了，因为通过构造器注入参数了
        // person.setName("tony");
        System.out.println(person.getName());
        System.out.println(person.getAvatar());
    }
}
```

#### set方式传参

除了通过构造器传参，我们还可以写`set`函数来传参，比如`setName`和`setAvatar`。


```java
class Avatar{
    private String url;
    // public Avatar(String url) {
    //     this.url = url;
    // }

    //增加set函数
    public void setUrl(String url) {
        this.url = url;
    }
}

class Person{
    private String name;

    private Avatar avatar;

    // public Person(String name, Avatar avatar) {
    //     this.name = name;
    //     this.avatar = avatar;
    // }

    public void setName(String name) {
        this.name = name;
    }

    public String getName() {
        return this.name;
    }

    public void setAvatar(Avatar avatar) {
        this.avatar = avatar;
    }

    public String getAvatar() {
        return this.avatar.url;
    }
}

```

修改XML配置文件，不在使用`constructor-arg`标签，而是换成`property`。不过除了标签名变了，其他的属性`name`,`value`,`ref`都是不变的。

```xml

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="avatar" class="com.test.java.Avatar">
         <property name="url" value="http://baidu.com"></property>
    </bean>

    <bean id="person" class="com.test.java.Person">
       <property name="name" value="tony"></property>
       <property name="avatar" ref="avatar"></property>
    </bean>

</beans>

```

执行一下`main`函数。

```java
public class main{
    
    public static void main(String[] args) {
        //这里先加载我们的XML配置文件
        BeanFactory ac = new ClassPathXmlApplicationContext("bean.xml");
        //通过容器获取一个对象,第一个参数也就是配置的id,第二个就是类
        Person person = ac.getBean("person", Person.class);
        // 这里不需要setName了，因为通过构造器注入参数了
        // person.setName("tony");
        System.out.println(person.getName());
        System.out.println(person.getAvatar());
    }
}
```

### 注解方式

注解方式要比`XML`方式简单的多，其中原理就是不再手动配置，而是通过注解告诉`Spring`我是一个`bean`。快来注册我吧。

主要是这4个注解告诉`Spring`

- @Component 单纯的说我是一个`bean`
- @Service 和上面的一样，不过一般用在service类中，更加语义化
- @Controller 和上面的一样，一般用在controller类中
- @Repository 我也是一个`bean`


接下来我们告诉`Spring`，你需要扫描出所有带上面注解的类，把他们注册到容器中。这一步需要修改XML文件,需要配置`<context:component-scan>`标签，并且通过`base-package`属性告诉`Spring`我们要扫描哪个目录

```xml

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="com.test.java"></context:component-scan>

</beans>

```

在类上面增加注解

```java

@Component
class Avatar{
    private String url;
    // public Avatar(String url) {
    //     this.url = url;
    // }

    //增加set函数
    public void setUrl(String url) {
        this.url = url;
    }
}

@Component
class Person{
    private String name;

    private Avatar avatar;

    // public Person(String name, Avatar avatar) {
    //     this.name = name;
    //     this.avatar = avatar;
    // }

    public void setName(String name) {
        this.name = name;
    }

    public String getName() {
        return this.name;
    }

    public void setAvatar(Avatar avatar) {
        this.avatar = avatar;
    }

    public String getAvatar() {
        return this.avatar.url;
    }
}

```

还可以自己指定扫描哪些注解，通过`context:include-filter`标签来指定。`type`类型写注解，`expression`指定扫描哪个注解。把标签放在`context:component-scan`这个里面就可以了。还需要在`context:component-scan`标签中指定，禁用默认的扫描方式。指定`use-default-filters`的属性为false.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="com.test.java" use-default-filters="false">
        <context:include-filter type="annotation" expression="org.springframework.stereotype.Component" />
    </context:component-scan>

</beans>
```

还可以排除一些注解不进行扫描，通过`context:exclude-filter`标签来指定。`type`同样写注解，`expression`指定排除的注解。


```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="com.test.java" use-default-filters="false">
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Component" />
    </context:component-scan>

</beans>
```

把类注册到容器中以后，我们还需要在使用的时候告诉容器，我们需要从容器中获取这个类，有5个注解

- @Autowired  Spring提供的，基于类型注入的，可以放在setter方法上
- @Qualifier  Spring提供的，基于名称注入的，一般和@Autowired配合使用来通过value参数指定名称
- @Resource   Java提供的，可以基于类型或名称注入的,可以通过name参数来指定名称，可以放在setter方法上
- @RequiredArgsConstructor  lombok提供的，基于类型注入，通过增加一个构造函数来注入。
- @Value      Spring提供的，注入基本类型的注解，一般用来从配置文件取值。


`@RequiredArgsConstructor`是lombok提供的，兼容性较差，像写单元测试的时候就用不了，它会给你的类增加一个构造方法，而且只会给`final`类型的属性进行注入。

```JAVA

@Component
//增加注解
@RequiredArgsConstructor
class Person{
    private String name;

    //错误使用，因为没有final
    // private Avatar avatar;

    //正确使用，加上final
    private final Avatar avatar;

    // public Person(String name, Avatar avatar) {
    //     this.name = name;
    //     this.avatar = avatar;
    // }

    public void setName(String name) {
        this.name = name;
    }

    public String getName() {
        return this.name;
    }

    public void setAvatar(Avatar avatar) {
        this.avatar = avatar;
    }

    public String getAvatar() {
        return this.avatar.url;
    }
}
```

这个时候可以编译完以后查看`.class`文件，看到的是这样的


```JAVA

@Component
// 注解没有了
// @RequiredArgsConstructor
class Person{
    private String name;

    //错误使用，因为没有final
    // private Avatar avatar;

    //正确使用，加上final
    private final Avatar avatar;

    //增加了一个构造函数
    public Person(final Avatar avatar) {
        this.avatar = avatar;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getName() {
        return this.name;
    }

    public void setAvatar(Avatar avatar) {
        this.avatar = avatar;
    }

    public String getAvatar() {
        return this.avatar.url;
    }
}
```

`@Autowired`是spring提供的，在spring中不管是写业务还是写单元测试都可以使用，它可以放在要注入的属性上面，也可以放在setter方法上面。使用他的时候不需要`final`修饰。

```JAVA

@Component
class Person{
    private String name;

    //在需要的属性上面增加这个注解，不需要final修饰
    @Autowired
    private Avatar avatar;

    //错误使用，加上final
    // @Autowired
    // private final Avatar avatar;

    // public Person(String name, Avatar avatar) {
    //     this.name = name;
    //     this.avatar = avatar;
    // }

    public void setName(String name) {
        this.name = name;
    }

    public String getName() {
        return this.name;
    }

    //也可以加在setter方法上面
    @Autowired
    public void setAvatar(Avatar avatar) {
        this.avatar = avatar;
    }

    public String getAvatar() {
        return this.avatar.url;
    }
}
```


`@Qualifier`注解配合`@Autowired`使用，比如我们有一个头像的接口

```java

interface IAvatar {
    String getUrl();
}

class maleAvatar implements IAvatar{
    public String getUrl() {
        return "male avatar";
    }
}

class femaleAvatar implements IAvatar{
    public String getUrl() {
        return "female avatar";
    }
}

```

这个时候我们在注入的时候如果只根据`IAvatar`来注入，容器就不知道我们需要哪个实现类了，所以我们需要指定类名.

```java

@Component
class Person{
    private String name;

    //在需要的属性上面增加这个注解，不需要final修饰
    @Autowired
    //指定要注入的实现类
    @Qualifier(value="maleAvatar")
    private IAvatar avatar;

    //错误使用，加上final
    // @Autowired
    // private final Avatar avatar;

    // public Person(String name, Avatar avatar) {
    //     this.name = name;
    //     this.avatar = avatar;
    // }

    public void setName(String name) {
        this.name = name;
    }

    public String getName() {
        return this.name;
    }

    //也可以加在setter方法上面
    @Autowired
    @Qualifier(value="maleAvatar")
    public void setAvatar(IAvatar avatar) {
        this.avatar = avatar;
    }

    public String getAvatar() {
        return this.avatar.url;
    }
}
```

`@Resource`更像是上面两个的合体，并且是由java提供的。也是可以放在属性和setter上面，并且不需要final修饰。

```java

@Component
class Person{
    private String name;

    //在需要的属性上面增加这个注解，不需要final修饰
    @Resource
    private Avatar avatar;

    //错误使用，加上final
    // @Resource
    // private final Avatar avatar;

    public void setName(String name) {
        this.name = name;
    }

    public String getName() {
        return this.name;
    }

    //也可以加在setter方法上面
    @Resource
    public void setAvatar(Avatar avatar) {
        this.avatar = avatar;
    }

    public String getAvatar() {
        return this.avatar.url;
    }
}
```

同样的，如果我们有多个实现类，需要指定可以通过它的`name`参数来指定。比如

```java

@Component
class Person{
    private String name;

    //在需要的属性上面增加这个注解，不需要final修饰
    @Resource(name="femaleAvatar")
    private IAvatar avatar;

    //错误使用，加上final
    // @Resource
    // private final Avatar avatar;

    public void setName(String name) {
        this.name = name;
    }

    public String getName() {
        return this.name;
    }

    //也可以加在setter方法上面
    @Resource(name="femaleAvatar")
    public void setAvatar(IAvatar avatar) {
        this.avatar = avatar;
    }

    public String getAvatar() {
        return this.avatar.url;
    }
}
```

`@Value`可以注入基本类型，比如字符串这种，但是更多的是从配置文件中取值。比如

```java
@Component
class Person{
    //直接注入tony字符串到name中
    @Value("tony")
    private String name;

    //从配置文件中取值
    //person:
    //  name: tony
    @Value("${person.name}")
    private String nameFromConfig;

    //在需要的属性上面增加这个注解，不需要final修饰
    @Resource(name="femaleAvatar")
    private IAvatar avatar;

    //错误使用，加上final
    // @Resource
    // private final Avatar avatar;

    public void setName(String name) {
        this.name = name;
    }

    public String getName() {
        return this.name;
    }

    //也可以加在setter方法上面
    @Resource(name="femaleAvatar")
    public void setAvatar(IAvatar avatar) {
        this.avatar = avatar;
    }

    public String getAvatar() {
        return this.avatar.url;
    }
}
```
