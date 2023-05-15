---
title: 转语言者速学java
date: 2023-03-02 10:12:47
tags: ['java','php']
category: java
article: 转语言者速学java
---

# 转语言者速学java

### 接口的默认方法

java的接口里面可以写方法体，这样的话，所有实现了该接口的，都自动具有了该方法而不用实现方法内容。当然了，也可以重写方法。

```java
public interface IGuideService {

    public default Integer getNum() {
        return 1;
    }
}

public class GuideService implements IGuideService {


}

public class Main {

    public static void main(String [] args) {
        IGuideService guideService = new GuideService;
        System.out.println(guideService.getNum());
    }
}
```


但是java可以实现多个接口，所以就可能会有`二义性`的问题，C++解决问题的方法是`虚基类`，java则是简单粗暴的。
- 如果是父类中和接口中的方法冲突了。那么选择父类中的方法。
- 如果是多个接口中的方法冲突了，那么你必须通过覆盖该方法来手动解决冲突。

看一下如果父类中也有`getNum`方法。创建一个父类返回2.再次执行以后会发现返回2了。

```java
public class BaseGuideService {

    public Integer getNum() {
        return 2;
    }
}

// 继承父类
public class GuideService extends BaseGuideService implements IGuideService {


}
```

接下来不用父类，然后创建两个接口试试。

```java
public interface IGuideNewService {

    public default Integer getNum() {
        return 3;
    }
}

// 实现两个接口
public class GuideService implements IGuideService,IGuideNewService {


}
```

这个时候编译就会报错了。
> GuideService 从 IGuideService 和 IGuideNewService 中继承了getNum() 的不相关默认值

那如果这两个接口是父子接口呢？这个时候就不会报错了，因为相当于子接口重写了这个方法。所以只存在一个`getNum`方法了。

我们来看如果是单独的两个接口，那么需要我们手动来解决冲突。重写一下这个方法。

```java
// 实现两个接口
public class GuideService implements IGuideService, IGuideNewService {

    @Override
    public Integer getNum() {
        return 5;
    }
}

```

在重写的时候也可以选择使用两个接口的某一个。

```java
// 实现两个接口
public class GuideService implements IGuideService, IGuideNewService {

    @Override
    public Integer getNum() {
        return IGuideService.super.getNum();

        //或者使用另一个
        //return IGuideNewService.super.getNum();
    }
}

```


### 日期时间

在java中的时间，有几个概念
- 某一个具体的时间点：Instant
- 持续时间，两个时间点之间的时间：Duration
- 本地时间，没有时区信息的：LocalDate,LocalTime,LocalDateTime
- 带时区的时间：ZonedDateTime
- 处理时区信息时，时间段使用 Period 而不是 Duration
- 时间格式化处理： DateTimeFormatter
- 日历计算，比如查找月份的第一天，第二天，第一周，第二周的星期二：TemporalAdjuster


获取当前的一个时间点。

```java
Instant.now();
```

获取一段时间，可以用来求运行速度。start是处理开始的时间点。end是处理完成的时间点。再通过Duration来求出一段时间.

```java
Instant start = Instant.now();
//处理逻辑
Instant end = Instant.now();

Duration time = Duration.between(start,end);
long mills = time.toMills();
```

获取本地当前时间

```java
LocalDateTime time = LocalDateTime.now();
```

时间格式化

```java
LocalDateTime time = LocalDateTime.now();
time.format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
```

获取带时区的当前时间

```java
ZonedDateTime time = ZonedDateTime.now();
```

想看更多时间处理的，可以参考 [LocalDateTime源码解析](https://thepatterraining.github.io/JAVA-spring%E5%B8%B8%E7%94%A8%E5%B7%A5%E5%85%B7%E6%80%BB%E7%BB%93.html#valine-comments)


### stream

stream可以帮我们处理很多东西。比如

|方法名| 说明 |
|:----|:----|
| map |  循环。可以简单理解为foreach |
| flatMap | 将二维数据展开成一维 |
| filter | 过滤数据 |
| distinct | 去重 |
| sorted | 排序 |
| limit | 限制只取n个元素 |
| skip | 跳过n个元素 |

除了这些还可以实现比如把list转换成逗号分隔的字符串。把list经过处理以后转换成set或者map。

还可以变成指定值为key,指定值为value的map。比如id为key，name为值或者map为值的map。

具体的可以参考下面三个文章。
- [stream数据结构和原理]()
- [stream的map,filter,sorted等使用和源码]()
- [stream的collect,foreach等使用和源码]()

### optional

java中的optional可以帮助你避免`空指针异常`.

具体的可以参考[optional并非银弹](https://thepatterraining.github.io/JAVA-optional%E5%B9%B6%E9%9D%9E%E9%93%B6%E5%BC%B9.html)

### 函数式编程和lambda



### 注解

- @Autowired  Spring提供的，基于类型注入的，可以放在setter方法上，变量上，构造函数上
- @Inject     同@Autowired
- @Qualifier  Spring提供的，基于名称注入的，一般和@Autowired配合使用来通过value参数指定名称
- @Name       同@Qualifier，给@Inject配合使用
- @Primary
- @Resource   Java提供的，可以基于类型或名称注入的,可以通过name参数来指定名称，可以放在setter方法上
- @RequiredArgsConstructor  lombok提供的，基于类型注入，通过增加一个构造函数来注入。
- @Value      Spring提供的，注入基本类型的注解，一般用来从配置文件取值。
- @Component 单纯的说我是一个`bean`
- @Service 和上面的一样，不过一般用在service类中，更加语义化
- @Controller 和上面的一样，一般用在controller类中
- @Repository 我也是一个`bean`，一般用在dao数据访问层
- @Bean 我也是一个bean，导入第三方包里面的注解
- @Import 导入组件
- @ImportSelector 返回需要导入的组件的全类名数组
- @ImportBeanDefinitionRegistrar 手动注册Bean到容器
- @JsonIgnore  json的时候忽略属性
- @Bean(initMethod="init",destoryMethod="destory") 设置初始化和销毁方法
- @PostConstruct：初始化方法
- @PreDestory：销毁方法
- BeanPostProcessor：bean的后置处理器，在bean初始化前后进行一些处理工作
- @Configuration 声明为配置类
- @ComponentScan 扫描Component
- @Aspect 声明一个切面
- @After 在方法执行之后执行（方法上）
- @Before 在方法执行之前执行（方法上）
- @Around 在方法执行之前与之后执行（方法上）
- @PointCut 声明切点
- @EnableAspectJAutoProxy 启Spring对AspectJ代理的支持
- @Profile 指定组件在哪个环境的情况下才能被注册到容器中，不指定，任何环境下都能注册这个组件。
- @Conditional 通过实现Condition接口，并重写matches方法，从而决定该bean是否被实例化。
- @EnableAsync 配置类中通过此注解开启对异步任务的支持
- @Async  在实际执行的bean方法使用该注解来声明其是一个异步任务
- @EnableScheduling  在配置类上使用，开启计划任务的支持
- @Scheduled  来申明这是一个任务，包括cron,fixDelay,fixRate等类型
- @EnableConfigurationProperties：开启对@ConfigurationProperties注解配置Bean的支持；
- @EnableJpaRepositories：开启对SpringData JPA Repository的支持；
- @EnableTransactionManagement：开启注解式事务的支持；
- @EnableCaching：开启注解式的缓存支持；
- @EnableAspectAutoProxy：开启对AspectJ自动代理的支持；
- @EnableWebMvc   在配置类中开启Web MVC的配置支持。
- @RequestMapping   用于映射web请求，包括访问路径和参数。
- @ResponseBody    支持将返回值放到response内，而不是一个页面，通常用户返回json数据。
- @RequestBody           允许request的参数在request体中，而不是在直接连接的地址后面。
- @PathVariable         用于接收路径参数，比如@RequestMapping(“/hello/{name}”)声明的路径，将注解放在参数前，即可获取该值，通常作为Restful的接口实现方法。
- @RestController       该注解为一个组合注解，相当于@Controller和@ResponseBody的组合，注解在类上，意味着，该Controller的所有方法都默认加上了@ResponseBody。
- @ControllerAdvice     全局异常处理 全局数据绑定 全局数据预处理
- @ExceptionHandler     用于全局处理控制器里的异常。
- @InitBinder           用来设置WebDataBinder，WebDataBinder用来自动绑定前台请求参数到Model中。
- @ModelAttribute       
- @Transactional 
    - name 当在配置文件中有多个 TransactionManager , 可以用该属性指定选择哪个事务管理器。
    - propagation 事务的传播行为，默认值为 REQUIRED。
        - REQUIRED  如果当前存在事务，则加入该事务，如果当前不存在事务，则创建一个新的事务。( 也就是说如果A方法和B方法都添加了注解，在默认传播模式下，A方法内部调用B方法，会把两个方法的事务合并为一个事务 ）
        - SUPPORTS  如果当前存在事务，则加入该事务；如果当前不存在事务，则以非事务的方式继续运行。
        - MANDATORY 如果当前存在事务，则加入该事务；如果当前不存在事务，则抛出异常。
        - REQUIRES_NEW  重新创建一个新的事务，如果当前存在事务，暂停当前的事务。
        - NOT_SUPPORTED 以非事务的方式运行，如果当前存在事务，暂停当前的事务。
        - NEVER         以非事务的方式运行，如果当前存在事务，则抛出异常。
        - NESTED        和 REQUIRED 效果一样。
    - isolation 事务的隔离度，默认值采用 DEFAULT
    - timeout   事务的超时时间，默认值为-1。如果超过该时间限制但事务还没有完成，则自动回滚事务。
    - read-only 指定事务是否为只读事务，默认值为 false；为了忽略那些不需要事务的方法，比如读取数据，可以设置 read-only 为 true。
    - rollback-for  用于指定能够触发事务回滚的异常类型，如果有多个异常类型需要指定，各类型之间可以通过逗号分隔。
    - no-rollback- for  抛出 no-rollback-for 指定的异常类型，不回滚事务。
- @Schema   表示此类对应的数据库表对应的schema。
- @JsonFormat   可以方便的把Date类型直接转化为我们想要的模式
- @Transient    如果一个属性并非数据库表的字段映射，就务必将其标示为@Transient
- @JsonProperty 可以指定某个属性和json映射的名称
- @Scope设置类型包括：

设置Spring容器如何新建Bean实例（方法上，得有@Bean）

① Singleton

（单例,一个Spring容器中只有一个bean实例，默认模式）,

② Protetype

（每次调用新建一个bean）,

③ Request
 
（web项目中，给每个http request新建一个bean）,

④ Session

（web项目中，给每个http session新建一个bean）,

⑤ GlobalSession

（给每一个 global http session新建一个Bean实例）