---
title: 用户状态（UserStatus） 和 用户类型（UserType） 应该如何在领域模型中表达？
date: 2025-07-18 10:12:47
tags: ['架构','DDD']
category: mysql
article: 用户状态（UserStatus） 和 用户类型（UserType） 应该如何在领域模型中表达？
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

# 用户状态（UserStatus） 和 用户类型（UserType） 应该如何在领域模型中表达？

这是一个很有代表性的建模问题：用户状态（UserStatus） 和 用户类型（UserType） 应该如何在领域模型中表达？

✅ 最佳实践结论（DDD 建模角度）：

> ✅ 建议建模为单独的值对象（Value Object），而不是直接使用 Integer 类型。

接下来分析一下为什么

✅ 原因分析：为什么不用普通的 Integer？

| 问题      | 使用 Integer 的弊端                 |
| ------- | ------------------------------ |
| 可读性差    | `status == 1` 到底代表啥？新手难以理解     |
| 安全性差    | 任何地方都可以传入不合法值，如 `status = 999` |
| 违反封装    | `User` 实体暴露了状态内部的具体编码实现        |
| 业务语义不明确 | `Integer` 不能表达“启用/冻结/注销”等领域含义  |

✅ 为什么用值对象更合适？

在 DDD 中，值对象（Value Object）是具有领域语义但无身份标识的对象，适合表达用户的“状态”、“类型”、“级别”等固定枚举值的场景。

它能提供：

✅ 明确的领域语言（如：UserStatus.ACTIVE、UserType.ADMIN）
✅ 不变性（状态是不可变的值对象）
✅ 合法性控制（通过私有构造 + 工厂控制范围）
✅ 方便扩展（可以加业务方法，如 canLogin()）

将用户状态设置为值对象以后，我们能明确的表达状态的含义，是`正常`、`冻结`还是`封禁`等等。且避免了非法值。并且如果要变更的话，是变更整个值对象。

在领域建模中，你可以一下子明白这个实体中这个值的含义。

在DDD中，不要使用传统的`数据建模`思想，而是要使用`领域建模`思想。

传统的数据建模思想的话，用户状态存储在数据库中通常都是1,2,3,4这样的Int值。

所以实体中一般也直接使用Int类型的值作为存储。

而`领域建模`思想则强调将真实世界中的业务映射成领域模型。数据库仅仅是领域模型`持久化`的一个地方而已。

✅ 示例：建模用户状态为值对象

```java
package com.example.accountbook.domain.model.user;

import java.util.Objects;

public class UserStatus {

    private final int code;

    public static final UserStatus ACTIVE = new UserStatus(1);
    public static final UserStatus FROZEN = new UserStatus(2);
    public static final UserStatus DELETED = new UserStatus(3);

    private UserStatus(int code) {
        this.code = code;
    }

    public static UserStatus of(int code) {
        return switch (code) {
            case 1 -> ACTIVE;
            case 2 -> FROZEN;
            case 3 -> DELETED;
            default -> throw new IllegalArgumentException("非法用户状态: " + code);
        };
    }

    public boolean isActive() {
        return this == ACTIVE;
    }

    public int getCode() {
        return code;
    }

    // equals / hashCode for value object identity

    @Override
    public boolean equals(Object o) {
        return (o instanceof UserStatus that) && this.code == that.code;
    }

    @Override
    public int hashCode() {
        return Objects.hash(code);
    }
}

```

聚合中使用值对象

```java
public class User {
    private Long id;
    private String username;
    private String passwordHash;

    private UserStatus status;  // ✅ 状态是值对象
    private UserType type;      // ✅ 类型是值对象

}

```

## 用 enum 枚举 + 值对象混合的高级方案

一般来说，用户状态和用户类型，我们都会创建一个枚举类。因此我们可以使用枚举+值对象的方式来重新构建一下用户状态。

我们现在深入 DDD 实践：用 Enum 枚举 + 值对象（Value Object）混合建模方式 表达如用户状态 / 用户类型，并展示如何在 MyBatis 中映射为字段、在 Repository 中自动转换为领域值对象。

✅ 为什么用「Enum + 值对象」组合建模？

| 对象类型           | 作用                          |
| -------------- | --------------------------- |
| `enum`         | 表达离散枚举值（常量集合）               |
| `Value Object` | 表达领域语义 + 方法（如 `isActive()`） |


> 🌟 组合起来，既有类型安全、又有语义表达、也便于持久化。

✅  枚举类：UserStatusEnum

```java
public enum UserStatusEnum {
    ACTIVE(1, "启用"),
    FROZEN(2, "冻结"),
    DELETED(3, "注销");

    private final int code;
    private final String label;

    UserStatusEnum(int code, String label) {
        this.code = code;
        this.label = label;
    }

    public int getCode() {
        return code;
    }

    public static UserStatusEnum fromCode(int code) {
        for (UserStatusEnum status : values()) {
            if (status.code == code) return status;
        }
        throw new IllegalArgumentException("未知状态: " + code);
    }
}
```

✅  值对象：UserStatus

```java
public class UserStatus {

    private final UserStatusEnum status;

    private UserStatus(UserStatusEnum status) {
        this.status = status;
    }

    public static UserStatus of(UserStatusEnum status) {
        return new UserStatus(status);
    }

    public static UserStatus of(int code) {
        return new UserStatus(UserStatusEnum.fromCode(code));
    }

    public boolean isActive() {
        return this.status == UserStatusEnum.ACTIVE;
    }

    public int getCode() {
        return this.status.getCode();
    }

    public String getLabel() {
        return this.status.name();
    }

    public UserStatusEnum getStatusEnum() {
        return this.status;
    }

    @Override
    public boolean equals(Object o) {
        return (o instanceof UserStatus that) && this.status == that.status;
    }

    @Override
    public int hashCode() {
        return status.hashCode();
    }
}

```

✅  MyBatis 持久化对象（PO）：UserPO
```java
public class UserPO {
    private Long id;
    private String username;
    private String passwordHash;
    private Integer status; // 状态值字段
    // ... 省略 getter/setter
}

```

✅  Mapper XML 映射
```xml
<mapper namespace="com.example.repository.mybatis.UserMapper">
    <select id="selectByUsername" resultType="com.example.repository.mybatis.UserPO">
        SELECT id, username, password_hash, status
        FROM users
        WHERE username = #{username}
    </select>
</mapper>
```

✅ 5. 仓储实现：UserRepositoryImpl → 转换 PO → 聚合

```java
@Repository
public class UserRepositoryImpl implements UserRepository {

    private final UserMapper mapper;

    public UserRepositoryImpl(UserMapper mapper) {
        this.mapper = mapper;
    }

    @Override
    public Optional<User> findByUsername(String username) {
        UserPO userPO = mapper.selectByUsername(username);
        return Optional.ofNullable(userPO).map(this::toEntity);
    }

    private User toEntity(UserPO doObj) {
        return new User(
            doObj.getId(),
            doObj.getUsername(),
            doObj.getPasswordHash(),
            UserStatus.of(doObj.getStatus()) // ← 关键转换点
        );
    }
}

```

✅  领域聚合实体：User

```java
public class User {
    private Long id;
    private String username;
    private String passwordHash;
    private UserStatus status; // 值对象

    public User(Long id, String username, String passwordHash, UserStatus status) {
        this.id = id;
        this.username = username;
        this.passwordHash = passwordHash;
        this.status = status;
    }

    public boolean canLogin() {
        return status.isActive();
    }

    public int getStatusCode() {
        return status.getCode();
    }

    public String getStatusLabel() {
        return status.getLabel();
    }

    // 省略其他 getter...
}

```

✅  保存数据时：实体 → PO
```java
UserPO userPO = new UserPO();
userPO.setId(user.getId());
userPO.setUsername(user.getUsername());
userPO.setPasswordHash(user.getPasswordHash());
userPO.setStatus(user.getStatus().getCode()); // ← 值对象 → DB 字段

```

✅ 总结对比

| 项       | `Integer` | `enum` | `Value Object + enum` ✅ 推荐 |
| ------- | --------- | ------ | -------------------------- |
| 类型安全    | ❌         | ✅      | ✅                          |
| 语义表达力   | ❌         | ⚠️ 限   | ✅                          |
| 是否可添加行为 | ❌         | ⚠️（有限） | ✅（如 `canLogin()`）          |
| 可维护性    | 差         | 一般     | 高                          |


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


