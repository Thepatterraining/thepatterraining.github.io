---
title: JAVA-optional并非银弹
date: 2023-03-02 10:12:47
tags: ['java','optional']
category: java
article: JAVA-optional并非银弹
---

# optional并非银弹

首先，我们都知道，`optional`类型要更加安全，如果使用的好，不会出现空指针异常，因为它不会返回null。

但是注意，这里的前提是`使用的好`。

比如 下面这两段代码。这里的 optionalInt.get().toString() 并不会比 num.toString() 安全，如果optionInt.get()返回的是一个null，还是会触发空指针异常。
```java
Optional<Integer> optionalInt = Optional.of(12);
optionalInt.get().toString();
```

```java
Integer num = 12;
num.toString();
```

所以，optional并不是处理空指针的银弹，而是需要正确的使用它。

### 如果正确的使用optional

`isPresent`和`ifPresent`这两个方法。
- isPresent 是一个判断，类似于 num != null
- ifPresent 接受一个 lambda 表达式或者方法，如果存在的话就调用该方法。

```java
Optional<Integer> optionalInt = Optional.of(12);
int num;
optionalInt.ifPresent(i -> {
    num = i;
});

if (optionalInt.isPresent()) {
    num = optionalInt.get();
}
```

这里更推荐的是使用 ifPresent 方法，更加安全方便。

为什么呢？因为你只是为了判断这么一下的话，完全可以使用 `num != null` 来代替 `optionalInt.isPresent`。毕竟这样还省去了包装optional的步骤，效果则是一样的。

```java
if (optionalInt.isPresent()) {
    num = optionalInt.get();
}

int a;
if (a != null) {
    num = a * 2;
}
```

但是 `ifPresent` 方法只负责处理，并不返回任何值。

如果你想要返回值的话，可以使用`map`方法代替。他返回一个bool值，被封装到optional中的true或者false(根据optionalInt是否存在)，也可能是个空值。

```java
Optional<Boolean> res = optionalInt.map(i -> {
    num = i;
});
```

那么在日常使用中，还会有默认值的情况，比如，如果int值存在我就赋值给num，不存在我就赋值0。这个时候就可以使用下面这三个方法
- orElse        如果有值，返回值，如果没有值，返回你给的默认值。
- orElseGet     和上面的效果一样，只是可以传一个lambda表达式
- orElseThrow   和上面的效果一样，没有值的时候返回一个异常。

```java
Optional<Integer> optionalInt = Optional.of(12);
num = optionalInt.orElse(0); //这里有值，所以返回12

Optional<Integer> optionalInt = Optional.empty();
num = optionalInt.orElse(0); //这里没有值，所以返回默认值0

Optional<Integer> optionalInt = Optional.empty();
// 传一个默认值方法
num = optionalInt.orElseGet(() -> {
    return 0;
});

Optional<Integer> optionalInt = Optional.empty();
// 如果没有值，返回一个异常
return optionalInt.orElseThrow(() -> {
        return new RuntimeException("异常了");
});
```

通过`faltMap`方法实现optional链式操作。首先通过of方法创建一个`Optional<Integer>`类型的12。然后通过flatMap方法把这个Integer的12传递给doubleInt方法。doubleInt方法处理完以后返回一个`Optional<Integer>`类型的24。

因为返回的还是一个`Optional`。所以还可以继续调用flatMap方法。将24传给intToStr方法。将24转换成String类型。然后返回一个`Optional<String>`类型的24.

`ofNullable`方法的作用是如果你给的值存在就调用`of`方法创建一个`Optional`。如果不存在就调用`empty`方法创建一个空的`Optional`。

```java

public String optionalMap() {
    Optional<String> res = Optional.of(12).flatMap(this::doubleInt).flatMap(this::intToStr);
    return res.get();
}

// 把一个数转换成string
public Optional<String> intToStr(int x) {
    return Optional.ofNullable(String.valueOf(x));
}

// 把一个数 * 2
public Optional<Integer> doubleInt(int x) {
    return Optional.ofNullable(x << 1);
}

```



