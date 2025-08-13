---
title: DDD记账软件实战三
date: 2025-08-08 10:12:47
tags: ['架构','DDD']
category: DDD
article: DDD记账软件实战三
---

> 大家好，我是大头，职高毕业，现在大厂资深开发，前上市公司架构师，管理过10人团队！
> 我将持续分享成体系的知识以及我自身的转码经验、面试经验、架构技术分享、AI技术分享等！
> 愿景是带领更多人完成破局、打破信息差！我自身知道走到现在是如何艰难，因此让以后的人少走弯路！
> 无论你是统本CS专业出身、专科出身、还是我和一样职高毕业等。都可以跟着我学习，一起成长！一起涨工资挣钱！
> 关注我一起挣大钱！文末有惊喜哦！

> 关注我发送“MySQL知识图谱”领取完整的MySQL学习路线。
> 发送“电子书”即可领取价值上千的电子书资源。
> 发送“大厂内推”即可获取京东、美团等大厂内推信息，祝你获得高薪职位。
> 发送“AI”即可领取AI学习资料。

# DDD记账软件实战三

## 前情提要

之前我们已经梳理了整体架构图：

![架构图](https://thepatterraining.github.io/images/ddd/ddd4-1.png)

除了我们一开始划分好的两个服务以外，还有一些支撑服务，属于不管干啥都需要用到的。

并且我们已经按照DDD划分了目录并创建了项目。
- starter: 接口层，包括HTTP接口、队列的消费者、DTO、启动类
- api: 接口层，提供RPC接口，包括外部RPC接口需要使用的DTO、枚举等
- application：应用服务层，放应用服务，负责编排领域服务、聚合根等。
- domain：领域服务层，放领域相关的一切信息，领域服务负责编排聚合根，聚合根负责完成自身的业务逻辑。
- infrastructure: 基础设施层，放配置、仓储、工厂、对外部的请求、发送MQ消息等。
- common: 放一些公共信息。

我们使用的版本如下：
- spring 3.5.4版本
- java21

## 如何设计一个登陆系统

开始一个应用程序当然是从`用户注册登陆`开始了。

现在的用户登陆一般都有很多种方式，比如：
- 用户名密码登录
- 手机号验证码登陆
- 微信扫码登陆
- 微信openId登陆
- 微信unionId登陆
- 邀请注册登录
- 其他第三方登录等等

面对如此多的登录方式，难道每次我们都要新加一个接口，并且去写一套登录逻辑吗？

有没有什么解决方案呢？

有的，兄弟，包有的。

### 业务流程

对于架构设计，我的理解就是`抽取通用的`，`扩展不通用的`。

而针对这些，在代码设计上已经有了很多前人的智慧了，比如开闭原则等，比如各种设计模式。

首先，我们应该梳理一下登陆的业务流程。万里长城始于脚下。不了解业务的开发不是一个好开发。

![登陆业务流程图](https://thepatterraining.github.io/images/ddd/ddd5-1.png)

首先， 用户在登录页面选择登录方式。接下来我们执行登录操作，如果登录成功，就生成Token并返回。如果登录失败，就返回错误信息。

到这里可以发现，除了登录方式不同，其他的流程基本都是一样的。

因此，我们应该把登录方式抽取出来。

### 业务架构

1. 侧重点
- 业务流程：展示业务活动的流动和交互，例如用户注册、订单处理、支付流程等。
- 业务实体：定义业务领域中的关键实体和它们之间的关系，例如用户、订单、产品等。
- 业务规则：描述业务逻辑和约束，例如折扣计算、库存校验等。
2. 目标
- 业务理解：帮助业务人员和开发团队理解业务需求和流程。
- 需求分析：作为需求分析和设计的基础，确保技术实现符合业务目标。
- 沟通协作：促进业务和技术团队之间的沟通和协作。

接下来我们需要分析，不同的登录方式都需要使用哪些能力。

对于用户名称密码登陆来说，简单的业务逻辑如下：
1. 校验参数
2. 根据用户名和密码查询用户信息
3. 校验用户状态、密码错误次数等
4. 生成Token
5. 写入登陆日志
6. 返回用户信息

对于手机号验证码登陆来说，简单的业务逻辑如下：
1. 校验参数
2. 校验验证码是否正确
3. 根据手机号查询用户信息
4. 校验用户状态等
5. 生成Token
6. 写入登陆日志
7. 返回用户信息

从这里，我们可以分析出共同的业务逻辑，比如：
- 校验参数
- 查询用户信息
- 校验用户是否能登陆
- 生成Token
- 写入登陆日志
- 返回用户信息

再看这里面需要用到的一些能力，比如：
- 生成Token
- 写入登陆日志
- 查询用户信息
- 发送验证码
- 校验验证码
- 风控

因此，我们可以梳理出下面的业务架构图

![登陆业务架构图](https://thepatterraining.github.io/images/ddd/ddd5-2.png)

当梳理出业务架构图以后，我们明确了本次业务中，需要使用哪些能力，还可以在业务架构图中通过不同的`颜色`标识出哪些业务能力是`已有`的，哪些业务能力是`本次建设`的。还有哪些能力可能是`以后建设`的。

这样，我们就明确了本次需要做哪些事情。

### 系统架构

1. 侧重点
- 技术组件：展示系统的技术组件和它们之间的交互，例如数据库、服务层、消息队列等。
- 系统模块：定义系统的模块化结构，例如用户服务、订单服务、支付服务等。
- 技术栈：描述技术栈和工具，例如编程语言、框架、数据库技术等。

2. 目标
- 技术实现：指导开发团队进行技术实现和系统构建。
- 性能优化：帮助识别系统性能瓶颈和优化机会。
- 系统维护：支持系统的维护和扩展，确保技术架构的可持续性。

我们明确了业务以后，接下来要将业务落地，如何实现业务也是比较重要的，这个时候我们就可以做出系统架构图。

系统架构图描述了登陆的具体技术组件，用哪些技术来实现登陆业务。

![登陆系统架构图](https://thepatterraining.github.io/images/ddd/ddd5-3.png)

### 详细设计

经过上面的架构设计以后，我们明确了下面几点：
1. 明确了本次需要实现哪些业务能力
2. 明确了这些业务能力的共同点和差异点
3. 明确了这些业务能力使用哪些技术实现

接下来，我们就需要明确如何实现了。这也就是详细设计需要做的内容。

比如我们可以通过用例图来理解需求。

![登陆系统用例图](https://thepatterraining.github.io/images/ddd/ddd5-4.png)

1. 核心用例
- 登录系统：主用例，用户的主要入口
- 使用 <<include>> 关系连接三种具体登录方式

2. 登录方式
- 用户名密码登录：传统登录方式
- 手机验证码登录：需要包含"发送验证码"子用例
- 微信OpenID登录：第三方社交登录

3. 扩展性设计
- 其他登录方式：通过 <<extend>> 关系预留扩展点
- 可以轻松添加如：
- 人脸识别登录
- 指纹登录
- 支付宝登录
- QQ登录等

4. 辅助功能
- 注册账号：新用户注册
- 找回密码：密码重置功能
- 发送验证码：被手机登录包含
- 验证身份：多种场景下的身份验证

可以通过UML类图来指导代码结构。

![登陆系统UML类图](https://thepatterraining.github.io/images/ddd/ddd5-5.png)

这些类使用了以下设计模式和原则：
- 策略模式：将不同的认证方式抽象为策略，便于扩展新的登录方式
- 工厂模式：通过工厂统一管理和创建认证策略
- 门面模式：提供统一的认证入口，隐藏内部复杂性
- 依赖倒置原则：高层模块依赖抽象而非具体实现
- 开闭原则：对扩展开放，对修改关闭

架构优势：
- 易于扩展新的登录方式
- 各认证策略相互独立，互不影响
- 统一的认证接口，使用简单
- 符合DDD领域设计思想

通过这些UML类图可以指导我们如何进行开发。

### 开发

当设计完成以后就可以进入开发阶段了。这里给出一些开发伪代码。

接口层：入口的Controller:
- AuthController
    - login: 登陆方法，所有登陆都走这个方法。

```java
public class AuthController{
    @Resource
    private AuthenticationFacade facade;

    public AuthResponse login(AuthRequest req) {
        //1. 参数校验
        validate(req);
        // 2. 调用登陆门面进行登陆
        AuthResponse res = facade.authenticate(req);
        // 3. 返回用户信息
        return res;
    }
}

```

应用服务层：编排登陆领域服务实现登陆逻辑
- AuthenticationFacade： 登陆门面，屏蔽登陆细节。
    - authenticate: 登陆方法,执行登陆操作
- AuthStrategyFactory： 登陆策略工厂，用来创建不同的登陆策略。
    - getStrategy：获取对应的登陆策略
    - register： 注册登陆策略

```java
public class AuthenticationFacade{

    @Resource
    private AuthStrategyFactory factory;

    public AuthResponse authenticate(AuthRequest req) {
        // 1. 通过工厂获取对应的策略
        AuthStrategy strategy = factory.getStrategy(req.getType);
        // 2. 构建请求参数
        AuthContext ctx = new AuthContext();
        // 3. 执行对应的登陆策略
        AuthResult result = strategy.authenticate(ctx);
        // 4. 返回结果
        AuthResponse res = convert(result);
        return res;
    }
}

public class AuthStrategyFactory{
    private Map<AuthType, AuthStrategy> loginStrategyMap;

    @Resource
    private List<AuthStrategy> loginStrategies;

    @PostConstruct
    public void init() {
        // 初始化的时候注册登陆策略
        loginStrategies.forEach(loginStrategy -> register(loginStrategy.supports(), loginStrategy));
    }

    public AuthStrategy getStrategy(AuthType type) {
        // 返回策略
        // 这里还需要判断如果没有实现这个策略怎么处理
        return loginStrategyMap.get(type);
    }

    public void register(AuthType type, AuthStrategy strategy) {
        loginStrategyMap.put(type, strategy);
    }

}
```

领域服务层：实现登陆逻辑
- AuthContext：登陆上下文信息，是一个值对象
- AuthType：登陆枚举
- AuthStrategy：登陆策略接口，所有的登陆策略都要实现这个接口
    - supports：登陆策略支持的登陆枚举
    - authenticate: 登陆策略的具体实现逻辑
- UsernamePasswordStrategy：用户名密码登陆策略，实现对应逻辑
- SmsCodeStrategy：手机号验证码登陆策略
- WechatOpenIdStrategy：微信OpenId登陆策略

```java
public interface AuthStrategy{
    AuthType supports();
    AuthResult authenticate(AuthContext ctx);
}

// 实现用户名密码登陆策略
public class UsernamePasswordStrategy implements AuthStrategy{
    public AuthType supports() {
        return AuthType.USERNAME_PASSWORD;
    }

    public AuthResult authenticate(AuthContext ctx) {
        // 处理逻辑
    }
}

// 实现手机号验证码登陆逻辑
public class SmsCodeStrategy implements AuthStrategy{
    public AuthType supports() {
        return AuthType.SMS;
    }

    public AuthResult authenticate(AuthContext ctx) {
        // 处理逻辑
    }
}

// 实现微信OpenId登陆逻辑
public class WechatOpenIdStrategy implements AuthStrategy{
    public AuthType supports() {
        return AuthType.WECHAT_OPENID;
    }

    public AuthResult authenticate(AuthContext ctx) {
        // 处理逻辑
    }
}
```

## 总结

我们从0-1，完整的实现了一个`可扩展`的登陆功能的设计，包括系统架构、业务架构、流程图、用例图、UML类图、以及最后开发落地。

整个设计思路如上所述。其他的系统我们同样可以根据这个思路进行设计。

## 文末福利

> 关注我发送“MySQL知识图谱”领取完整的MySQL学习路线。
> 发送“电子书”即可领取价值上千的电子书资源。
> 发送“大厂内推”即可获取京东、美团等大厂内推信息，祝你获得高薪职位。
> 发送“AI”即可领取AI学习资料。
> 部分电子书如图所示。

![概念学习](https://thepatterraining.github.io/images/bottom1.png)

![概念学习](https://thepatterraining.github.io/images/bottom2.png)

![概念学习](https://thepatterraining.github.io/images/bottom3.png)

![概念学习](https://thepatterraining.github.io/images/bottom4.png)


