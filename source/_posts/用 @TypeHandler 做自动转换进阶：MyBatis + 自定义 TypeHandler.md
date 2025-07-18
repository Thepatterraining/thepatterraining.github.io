---
title: 用 @TypeHandler 做自动转换	进阶：MyBatis + 自定义 TypeHandler
date: 2025-07-18 15:12:47
tags: ['架构','DDD']
category: mysql
article: 用 @TypeHandler 做自动转换	进阶：MyBatis + 自定义 TypeHandler
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

# 用 @TypeHandler 做自动转换进阶：MyBatis + 自定义 TypeHandler

之前讲过通过枚举+值对象的方式来构建用户状态和用户类型。

但是每次查询的时候我们都需要手动的将`数据库`中的1，2，3来映射成值对象。

我们可以使用`TypeHandler`来实现自动转换，就不需要每次都手动执行了。

## 场景目标

已经使用了 UserStatus 值对象 + UserStatusEnum 枚举来表达用户状态（如 ACTIVE、FROZEN、DELETED），现在希望：
- 数据库存储 Integer 类型（如：1、2、3）
- MyBatis 能自动将 Integer → UserStatus
- 不再每次手动 UserStatus.of(statusInt) 转换

## 最佳方案：MyBatis 自定义 @TypeHandler

✅ 1. 项目结构

```
domain.model.user
└── UserStatus.java（值对象）
└── UserStatusEnum.java（枚举）

infrastructure.mybatis.typehandler
└── UserStatusTypeHandler.java（自定义 TypeHandler）

infrastructure.repository.mybatis
└── UserMapper.java
└── UserPO.java
└── UserMapper.xml
```

✅ 2. 代码实现

✅ 值对象：UserStatus（简化版）

```java
public class UserStatus {
    private final UserStatusEnum status;

    public static final UserStatus ACTIVE = new UserStatus(UserStatusEnum.ACTIVE);
    public static final UserStatus FROZEN = new UserStatus(UserStatusEnum.FROZEN);
    public static final UserStatus DELETED = new UserStatus(UserStatusEnum.DELETED);

    private UserStatus(UserStatusEnum status) {
        this.status = status;
    }

    public static UserStatus of(int code) {
        return new UserStatus(UserStatusEnum.fromCode(code));
    }

    public int getCode() {
        return status.getCode();
    }

    public UserStatusEnum getStatusEnum() {
        return status;
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

✅ 枚举类：UserStatusEnum

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

    public int getCode() { return code; }

    public static UserStatusEnum fromCode(int code) {
        for (UserStatusEnum e : values()) {
            if (e.code == code) return e;
        }
        throw new IllegalArgumentException("未知状态码: " + code);
    }
}

```

✅ 自定义 TypeHandler：UserStatusTypeHandler
```java
@MappedJdbcTypes(JdbcType.INTEGER)
@MappedTypes(UserStatus.class)
public class UserStatusTypeHandler extends BaseTypeHandler<UserStatus> {

    @Override
    public void setNonNullParameter(PreparedStatement ps, int i, UserStatus parameter, JdbcType jdbcType) throws SQLException {
        ps.setInt(i, parameter.getCode());
    }

    @Override
    public UserStatus getNullableResult(ResultSet rs, String columnName) throws SQLException {
        int code = rs.getInt(columnName);
        return UserStatus.of(code);
    }

    @Override
    public UserStatus getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
        int code = rs.getInt(columnIndex);
        return UserStatus.of(code);
    }

    @Override
    public UserStatus getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {
        int code = cs.getInt(columnIndex);
        return UserStatus.of(code);
    }
}

```

✅ 3. PO 层配置 TypeHandler

```java
public class UserPO {
    private Long id;
    private String username;
    private String passwordHash;

    @TableField(typeHandler = UserStatusTypeHandler.class)
    private UserStatus status;

    // getter/setter
}

```

如果你用 XML 而不是注解，也可以在 XML 中配置：

```xml
<result column="status" property="status" typeHandler="com.example.infrastructure.mybatis.typehandler.UserStatusTypeHandler"/>
```

✅ 4. 注册 TypeHandler（推荐方式）
Spring Boot + MyBatis：
确保你在配置类里添加扫描：
```java
@MapperScan("com.example.infrastructure.repository.mybatis")
@Configuration
public class MyBatisConfig {
    @Bean
    public ConfigurationCustomizer configurationCustomizer() {
        return configuration -> {
            configuration.getTypeHandlerRegistry().register(UserStatusTypeHandler.class);
        };
    }
}

```

> 或者：使用 Spring Boot 自动注册（只要包结构正确 + 被 Spring 管理）

✅ 5. 使用效果

现在，MyBatis 自动完成：
- SELECT 时：从 int 字段转成 UserStatus 值对象
- INSERT/UPDATE 时：从 UserStatus 转为 int

你无需手动 `.of()` 转换，仓储代码更简洁：

```java
UserPO userPO = mapper.selectByUsername("jack");
User user = new User(
    userPO.getId(),
    userPO.getUsername(),
    userPO.getPasswordHash(),
    userPO.getStatus() // ✅ 直接是 UserStatus
);

```

## 总结

| 问题                   | 建议                                 |
| -------------------- | ---------------------------------- |
| 如何让 MyBatis 自动转换值对象？ | ✅ 自定义 `@TypeHandler` 并在字段或 XML 中指定 |
| 推荐封装格式？              | ✅ 使用 `enum + ValueObject` 组合建模     |
| 是否适用于其他值对象？          | ✅ 适用于所有 “数据库是整数/字符串” 的值对象场景        |


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


