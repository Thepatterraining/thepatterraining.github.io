---
title: DDD记账软件实战二
date: 2025-07-31 10:12:47
tags: ['架构','DDD']
category: DDD
article: DDD记账软件实战二
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

# DDD记账软件实战二

经过一开始的战略设计和战术设计以后，我们有了一个初步的服务划分
- 用户服务：管理用户相关的信息
- 记账服务：管理账本和收支相关的信息

并且，我们已经知道了我们要创建一个什么样的目录结构，每个服务的目录如下：
- starter: 接口层，包括HTTP接口、队列的消费者、DTO、启动类
- api: 接口层，提供RPC接口，包括外部RPC接口需要使用的DTO、枚举等
- application：应用服务层，放应用服务，负责编排领域服务、聚合根等。
- domain：领域服务层，放领域相关的一切信息，领域服务负责编排聚合根，聚合根负责完成自身的业务逻辑。
- infrastructure: 基础设施层，放配置、仓储、工厂、对外部的请求、发送MQ消息等。
- common: 放一些公共信息。

我们这个整体架构图如下：

![架构图](https://thepatterraining.github.io/images/ddd/ddd4-1.png)

除了我们一开始划分好的两个服务以外，还有一些支撑服务，属于不管干啥都需要用到的。

1. 网关

首先，需要通过网关来进行流量转发、权限校验、负载均衡等。
- 网关使用`Spring Cloud GateWay`来实现，网关不需要采用DDD来设计。

2. 缓存

缓存也是必不可少的一环，缓存使用`Redis`来实现。

3. 唯一ID生成

每个服务都需要使用到唯一ID，因此可以使用分布式唯一ID发号器。我们的案例里面使用的是`SnowFlake`算法进行生成，这个方案有一个缺点就是强依赖系统时钟。对于我们来说已经够用了。

如果需要更加强大的方案可以使用美团开源的`Leaf`来进行唯一ID生成，Leaf支持`号段模式`和`SnowFlake`模式，解决了SnowFlake强依赖系统时钟的问题。

4. 日志收集

日志收集也是必不可少的一环，我们日常都需要将日志统一收集到一起，并提供查询功能。我们直接使用ELK来实现。

5. 链路追踪

链路追踪也一样，毕竟多个微服务之间需要通过一个`唯一请求ID`来实现链路追踪，查询出一次请求的微服务以及对应的信息。
- 我们本次使用`SkyWalking`来实现链路追踪的能力。

6. 消息触达服务

不管做什么业务，其实都需要发送一些消息，包括短信消息、邮件消息、微信消息等等。

因此我们同样抽象出一个消息触达服务来执行消息发送功能。

## 创建用户服务

首先，我们要创建一个spring项目。可以直接使用[start.spring.io](https://start.spring.io/)来创建。

我们选择的是
- java
- maven
- 3.5.4版本
- java21
- Jar
 
接下来输入一些基础信息就可以了。

![创建用户服务](https://thepatterraining.github.io/images/ddd/ddd4-2.png)

点击下面的generate就可以把项目代码下载下来了。

下载下来以后，我们需要调整maven文件和目录结构。

最外层的maven文件内容如下，我们主要引入了下面的一些依赖
- spring cloud
- nacos: 配置管理和服务注册发现
- mysql
- mybatis plus
- jwt
- lombok

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>3.5.3</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.zt.bookkeeping.user</groupId>
	<artifactId>user</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>pom</packaging>
	<name>user</name>
	<description>记账软件用户服务</description>
	<url/>
	<licenses>
		<license/>
	</licenses>
	<developers>
		<developer/>
	</developers>
	<scm>
		<connection/>
		<developerConnection/>
		<tag/>
		<url/>
	</scm>
	<properties>
		<java.version>21</java.version>
		<spring-cloud.version>2025.0.0</spring-cloud.version>
		<spring-cloud-alibaba.version>2023.0.3.2</spring-cloud-alibaba.version>
		<project.version>0.0.1-SNAPSHOT</project.version>
	</properties>

	<modules>
		<module>user-domain</module>
		<module>user-application</module>
		<module>user-infrastructure</module>
		<module>user-starter</module>
		<module>user-common</module>
	</modules>

	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>

			<!-- 添加Spring Cloud Alibaba依赖管理 -->
			<dependency>
				<groupId>com.alibaba.cloud</groupId>
				<artifactId>spring-cloud-alibaba-dependencies</artifactId>
				<version>${spring-cloud-alibaba.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>

			<!-- 内部模块依赖管理 -->
			<dependency>
				<groupId>com.zt.bookkeeping.user</groupId>
				<artifactId>user-domain</artifactId>
				<version>${project.version}</version>
			</dependency>
			<dependency>
				<groupId>com.zt.bookkeeping.user</groupId>
				<artifactId>user-application</artifactId>
				<version>${project.version}</version>
			</dependency>
			<dependency>
				<groupId>com.zt.bookkeeping.user</groupId>
				<artifactId>user-infrastructure</artifactId>
				<version>${project.version}</version>
			</dependency>
		</dependencies>
	</dependencyManagement>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<!-- 添加Nacos服务发现依赖 -->
		<dependency>
			<groupId>com.alibaba.cloud</groupId>
			<artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
		</dependency>

		<!-- 添加Nacos配置管理依赖 -->
		<dependency>
			<groupId>com.alibaba.cloud</groupId>
			<artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
		</dependency>

		<!-- 添加bootstrap配置支持 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-bootstrap</artifactId>
		</dependency>

		<dependency>
			<groupId>com.mysql</groupId>
			<artifactId>mysql-connector-j</artifactId>
			<scope>runtime</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>

		<dependency>
			<groupId>com.baomidou</groupId>
			<artifactId>mybatis-plus-spring-boot3-starter</artifactId>
			<version>3.5.6</version>
		</dependency>
		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
			<version>1.18.30</version>
		</dependency>

		<!-- 引入jwt-->
		<dependency>
			<groupId>io.jsonwebtoken</groupId>
			<artifactId>jjwt-api</artifactId>
			<version>0.11.2</version>
		</dependency>
		<dependency>
			<groupId>io.jsonwebtoken</groupId>
			<artifactId>jjwt-impl</artifactId>
			<version>0.11.2</version>
			<scope>runtime</scope>
		</dependency>
		<dependency>
			<groupId>io.jsonwebtoken</groupId>
			<artifactId>jjwt-jackson</artifactId>
			<version>0.11.2</version>
			<scope>runtime</scope>
		</dependency>
	</dependencies>

</project>
```

调整了目录结构，整体目录如下图

![目录图](https://thepatterraining.github.io/images/ddd/ddd4-3.png)

我们首先复制一份代码放到目录下面并改名为`user-starter`。然后修改maven文件如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>com.zt.bookkeeping.user</groupId>
		<artifactId>user</artifactId>
		<version>0.0.1-SNAPSHOT</version>
	</parent>

	<artifactId>user-starter</artifactId>
	<packaging>jar</packaging>
	<name>user-starter</name>
	<description>用户服务启动模块</description>

	<dependencies>
		<!-- 依赖应用层 -->
		<dependency>
			<groupId>com.zt.bookkeeping.user</groupId>
			<artifactId>user-application</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-validation</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
			<version>1.18.12</version>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>
```

接下来用同样的方法复制出应用服务层。改名`user-application`.修改maven文件如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>com.zt.bookkeeping.user</groupId>
		<artifactId>user</artifactId>
		<version>0.0.1-SNAPSHOT</version>
	</parent>

	<artifactId>user-application</artifactId>
	<packaging>jar</packaging>
	<name>user-application</name>
	<description>用户服务应用层</description>

	<dependencies>
		<!-- 依赖领域层 -->
		<dependency>
			<groupId>com.zt.bookkeeping.user</groupId>
			<artifactId>user-domain</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>jakarta.validation</groupId>
			<artifactId>jakarta.validation-api</artifactId>
		</dependency>
	</dependencies>

</project>

```

在复制出领域层，改名`user-domain`。maven文件如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>com.zt.bookkeeping.user</groupId>
		<artifactId>user</artifactId>
		<version>0.0.1-SNAPSHOT</version>
	</parent>

	<artifactId>user-domain</artifactId>
	<packaging>jar</packaging>
	<name>user-domain</name>
	<description>用户服务领域层</description>

	<dependencies>
		<!-- 依赖common -->
		<dependency>
			<groupId>com.zt.bookkeeping.user</groupId>
			<artifactId>user-common</artifactId>
			<version>0.0.1-SNAPSHOT</version>
			<scope>compile</scope>
		</dependency>
		<!-- 领域层只需要基础的Java依赖，不需要Spring Web -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
		</dependency>
	</dependencies>

</project>

```

继续复制出基础设施层，改名`user-infrastructure`。maven文件如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>com.zt.bookkeeping.user</groupId>
		<artifactId>user</artifactId>
		<version>0.0.1-SNAPSHOT</version>
	</parent>

	<artifactId>user-infrastructure</artifactId>
	<packaging>jar</packaging>
	<name>user-infrastructure</name>
	<description>用户服务基础设施层</description>

	<dependencies>
		<!-- 依赖领域层 -->
		<dependency>
			<groupId>com.zt.bookkeeping.user</groupId>
			<artifactId>user-domain</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter</artifactId>
		</dependency>
		<dependency>
			<groupId>com.mysql</groupId>
			<artifactId>mysql-connector-j</artifactId>
			<scope>runtime</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>

		<dependency>
			<groupId>com.baomidou</groupId>
			<artifactId>mybatis-plus-spring-boot3-starter</artifactId>
			<version>3.5.6</version>
		</dependency>

		<!-- 引入jwt-->
		<dependency>
			<groupId>io.jsonwebtoken</groupId>
			<artifactId>jjwt-api</artifactId>
			<version>0.11.2</version>
		</dependency>
		<dependency>
			<groupId>io.jsonwebtoken</groupId>
			<artifactId>jjwt-impl</artifactId>
			<version>0.11.2</version>
			<scope>runtime</scope>
		</dependency>
		<dependency>
			<groupId>io.jsonwebtoken</groupId>
			<artifactId>jjwt-jackson</artifactId>
			<version>0.11.2</version>
			<scope>runtime</scope>
		</dependency>
	</dependencies>

</project>
```

最后，复制出common层，即可。maven文件如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>com.zt.bookkeeping.user</groupId>
		<artifactId>user</artifactId>
		<version>0.0.1-SNAPSHOT</version>
	</parent>

	<artifactId>user-common</artifactId>
	<packaging>jar</packaging>
	<name>user-common</name>
	<description>用户服务公共层</description>

	<dependencies>
	</dependencies>

</project>

```

### 设置启动类

我们将启动类放到`user-starter`下面。内容如下：

```java
package com.zt.bookkeeping.user;

import lombok.extern.slf4j.Slf4j;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.scheduling.annotation.EnableAsync;

@SpringBootApplication
@EnableAsync
public class UserApplication {

	private static final Logger logger = LoggerFactory.getLogger(UserApplication.class);

	public static void main(String[] args) {
		SpringApplication.run(UserApplication.class, args);
		logger.info("用户服务启动成功");
	}

}
```

目录如下图：

![启动用户服务](https://thepatterraining.github.io/images/ddd/ddd4-4.png)

接下来启动服务，启动成功，打印出`用户服务启动成功`。我们就成功创建了DDD结构的用户微服务了。

## 总结

本次我们梳理了整体的系统架构图，为接下来的工作做了准备，并且带大家`从0-1`创建了`用户微服务`。并且改造成了DDD项目的目录结构。

至于剩下的`记账微服务`。用同样的方式创建即可。遇到问题可以随时在评论区留言～

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


