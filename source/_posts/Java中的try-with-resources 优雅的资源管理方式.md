---
title: 还在手动管理资源？还在经常陷入OOM？一招教你快速解决
date: 2025-02-12 10:12:47
tags: ['Java']
category: Java
article: 还在手动管理资源？还在经常陷入OOM？一招教你快速解决
---

> 大家好，我是大头，职高毕业，现在大厂资深开发，前上市公司架构师，管理过10人团队！
> 我将持续分享成体系的知识以及我自身的转码经验、面试经验、架构技术分享、AI技术分享等！
> 愿景是带领更多人完成破局、打破信息差！我自身知道走到现在是如何艰难，因此让以后的人少走弯路！
> 无论你是统本CS专业出身、专科出身、还是我和一样职高毕业等。都可以跟着我学习，一起成长！一起涨工资挣钱！
> 关注我一起挣大钱！文末有惊喜哦！

# 还在手动管理资源？还在经常陷入OOM？一招教你快速解决

你是否还在为了手动管理资源而发愁？打开文件以后还要记得关闭文件，这些资源全部要手动管理，一旦忘了就有可能OOM！

一招教你使用`try-with-resources`来自动管理资源，不需要再手动关闭资源了，再也不怕忘记关闭资源了。

## try-with-resources

在Java开发中，资源管理一直是一个重要的问题。

无论是文件操作、数据库连接还是网络通信，都需要确保资源在使用后能够正确关闭，以避免资源泄漏和潜在的错误。

`try-with-resources`语句的引入，为Java开发者提供了一种简洁而优雅的资源管理方式，大大简化了代码的复杂性，同时也提高了代码的可读性和安全性。

## 传统资源管理的痛点

首先，我们来看看`try-with-resources`解决了什么问题。

1. 冗长的代码：传统资源管理的方式，通常需要在`finally`块里面手动关闭资源。但是这样既麻烦，又导致代码很多，而这些代码增加以后，比如多个资源关闭，嵌套关闭等，会导致可读性变差，代码难以管理。
2. 容易出错：当代码复杂、可读性差、难以管理以后，就很容易出错了。还有如果在关闭资源的时候抛出异常，那么资源可能就无法关闭了，从而导致资源泄漏问题。
3. 异常处理复杂：通常在`finally`中关闭资源的时候，还要在套一层`try...catch`，导致异常处理变多、复杂等问题

### 冗长的代码

在`try-with-resources`出现之前，Java开发者通常使用`try-finally`语句来确保资源被正确关闭。例如，当操作文件时，代码通常如下所示：

```java
FileInputStream inputStream = null;
try {
    inputStream = new FileInputStream("example.txt");
    // 读取文件内容
} catch (IOException e) {
    e.printStackTrace();
} finally {
    if (inputStream != null) {
        try {
            inputStream.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

这种写法虽然能够确保资源被关闭，但代码显得冗长且复杂。特别是当需要管理多个资源时，finally块中的关闭逻辑会变得更加繁琐。

### 容易出错

手动管理资源时，开发者需要在`finally`块中显式调用资源的`close()`方法。如果忘记关闭资源，或者在关闭资源时抛出异常而未正确处理，就可能导致资源泄漏或其他问题。例如：

```java
FileInputStream inputStream = new FileInputStream("example.txt");
// 使用inputStream读取文件内容
inputStream.close(); // 如果这里抛出异常，资源可能无法正确关闭
```

### 异常处理复杂

在`finally`块中关闭资源时，如果资源的`close()`方法抛出异常，而try块中也抛出了异常，处理这些异常会变得非常复杂。例如：

```java
FileInputStream inputStream = null;
try {
    inputStream = new FileInputStream("example.txt");
    // 读取文件内容，可能抛出异常
} catch (IOException e) {
    e.printStackTrace();
} finally {
    if (inputStream != null) {
        try {
            inputStream.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

如果try块和finally块都抛出异常，处理这些异常会变得非常复杂，容易导致错误。

## 什么是try-with-resources?

`try-with-resources`是`Java 7`引入的一种语法结构，用于`自动管理资源`。

它允许在try语句中声明一个或多个资源，这些资源必须实现了`AutoCloseable接口`或其子接口Closeable。

当try语句块执行完毕后，无论是否发生异常，`try-with-resources`都会自动调用资源的`close()`方法来关闭资源。

### 核心特性

- 自动关闭资源：try-with-resources会自动调用资源的close()方法，确保资源被正确关闭。
- 简化代码：无需手动在finally块中关闭资源，减少了代码冗余。
- 支持多个资源：可以同时管理多个资源，每个资源之间用分号隔开。
- 异常处理：如果资源的close()方法抛出异常，而try语句块中也抛出了异常，try语句块中抛出的异常会被传播，而资源关闭时抛出的异常会被抑制（suppressed）。

## 为什么使用try-with-resources

上面已经介绍了传统资源管理的痛点问题，所以使用`try-with-resources`主要就是解决这些痛点。

当然了，除了能解决这些问题以外，还有一些其他的好处。

毕竟，Java语言使用了这么长时间，能使它自身提供的一些特性，都是很有用的。

- 解决传统资源管理的痛点：解决上述的痛点问题。
- 提高代码的可读性和安全性：try-with-resources通过自动管理资源，简化了代码结构，提高了代码的可读性和安全性。它确保了资源在使用后能够正确关闭，减少了因资源泄漏导致的潜在问题。
- 支持自定义资源类：除了Java标准库中提供的资源类（如FileInputStream、FileOutputStream等），我们还可以自定义实现AutoCloseable接口的资源类，以便在try-with-resources中使用。

## 何时使用try-with-resources

### 使用场景

上面已经介绍了很多了，接下来看一下，有哪些使用场景。

- 文件操作：读取或写入文件时，使用FileInputStream、FileOutputStream等类。
- 数据库操作：使用Connection、Statement、ResultSet等资源时。
- 网络通信：使用Socket、ServerSocket等资源时。
- 自定义资源：任何实现了AutoCloseable接口的资源类。

### 注意事项

没有任何方法是一种`银弹`。技术方案没有黑魔法！所以在使用的时候也需要注意一些事项。

- 确保资源实现AutoCloseable接口：只有实现了AutoCloseable接口或Closeable接口的资源类才能在try-with-resources中使用。
- 异常处理：虽然try-with-resources会自动关闭资源，但仍然需要处理可能抛出的异常。
- 性能考虑：虽然try-with-resources简化了代码，但调用close()方法可能会增加一定的开销。在资源较多时，需要注意性能影响。

## 如何使用try-with-resources

经过上面的介绍，相信你已经对`try-with-resources`很了解了，接下来看一下到底该如何实战吧！

### 基本语法

`try-with-resources`语句允许在`try`块中声明一个或多个资源，这些资源必须实现了`AutoCloseable`接口或其子接口`Closeable`。当`try`块执行完毕后，无论是否发生异常，`try-with-resources`都会自动调用资源的`close()`方法来关闭资源。

```java
try (资源声明) {
    // 使用资源
} catch (异常类型 异常变量) {
    // 处理异常
}
```

### 示例代码

以下是一个使用try-with-resources读取文件内容的示例:

可以看到，我们把资源的获取放在了try后面的小括号里面，而不是以前一样放在了大括号里面。

这样的话，就不需要在`finally`里面关闭资源了，因为会自动关闭。

这也是因为`FileInputStream`实现了`Closeable`接口，因此可以作为资源在try-with-resources中声明。

```java
try (FileInputStream inputStream = new FileInputStream("example.txt")) {
    // 使用inputStream读取文件内容
} catch (IOException e) {
    e.printStackTrace();
}
```

### 管理多个资源

try-with-resources不仅可以管理单个资源，还可以同时管理多个资源。例如，当需要同时读取和写入文件时，可以这样写：

在这个例子中，inputStream和outputStream都被声明为资源，它们的close()方法都会在try语句块执行完毕后自动调用。

```java
try (FileInputStream inputStream = new FileInputStream("input.txt");
     FileOutputStream outputStream = new FileOutputStream("output.txt")) {
    // 使用inputStream读取文件内容并写入outputStream
} catch (IOException e) {
    e.printStackTrace();
}
```

### 自定义资源类

除了Java标准库中提供的资源类（如FileInputStream、FileOutputStream等），我们还可以自定义实现AutoCloseable接口的资源类，以便在try-with-resources中使用。例如：

只要我们自己的资源类，实现了`AutoCloseable`接口，就可以了。很简单吧！

```java
public class MyResource implements AutoCloseable {
    @Override
    public void close() throws Exception {
        System.out.println("资源已关闭");
    }
}
```

接下来就可以和上面一样的使用方法来使用我们自定义的资源类了！

```java
try (MyResource myResource = new MyResource()) {
    // 使用myResource
} catch (Exception e) {
    e.printStackTrace();
}
```

## 总结

`try-with-resources`是Java 7引入的一种非常有用的语法结构，

它能够自动管理资源，简化了资源关闭的代码，提高了代码的可读性和安全性。

通过实现AutoCloseable接口或Closeable接口，我们可以自定义资源类，并在try-with-resources中使用它们。

在实际开发中，合理使用try-with-resources可以有效避免资源泄漏，提高代码质量。

## 文末福利

> 关注我发送“MySQL知识图谱”领取完整的MySQL学习路线。
> 发送“电子书”即可领取价值上千的电子书资源。
> 发送“大厂内推”即可获取京东、美团等大厂内推信息，祝你获得高薪职位。
> 部分电子书如图所示。

![概念学习](https://thepatterraining.github.io/images/bottom1.png)

![概念学习](https://thepatterraining.github.io/images/bottom2.png)

![概念学习](https://thepatterraining.github.io/images/bottom3.png)

![概念学习](https://thepatterraining.github.io/images/bottom4.png)


