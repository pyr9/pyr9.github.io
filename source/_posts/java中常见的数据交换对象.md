---
title: java中常见的数据交换对象
date: 2025-07-29 21:09:00
tags:
categories: Java基础
---

# 1. 常见的数据交换对象

## 1. DTO

1. **目的**：用于服务层与外部系统（如前端、第三方服务）之间传输数据，通常是序列化的对象。
2. **特点**：
   - 常作为 Controller 接口的请求/响应参数
   - 可以包含多个PO的组合数据。
   - 字段不一定与数据库一致。
   - 无业务逻辑

举例：

```java
// 用于向前端封装用户登录信息
public class LoginDTO {
    private String username;
    private String password;
}
```

```java
// 用于向前端返回用户信息（不包含密码）
public class UserDTO {
    private Long id;
    private String username;
    private String nickName;
    private String avatar;
    private Integer age;
    private String roleName; // 关联角色名
}
```

## 2. BO 

1. **目的**：封装业务逻辑或业务规则，用于服务层内部处理。
2. **特点**：
   - 常在 Service 层中使用
   - 可包含业务方法和业务校验逻辑
   - 可能由多个DTO/PO组合而成，用于Service层处理复杂业务。

举例：

```java
// 订单业务对象
public class OrderBO {
    private String orderId;
    private BigDecimal amount;
    private Integer itemCount;
    private UserBO user; // 用户信息
    private List<OrderItemBO> items;
    private String discountRule;
    private BigDecimal finalAmount;
    private Boolean isOverdue;

    // 业务方法
    public void calculateTotal() { ... }
    public void applyDiscount() { ... }
}
```

```java
// User service 接口
public interface ISysUserService {
  boolean checkUserNameUnique(SysUserBo user);
  boolean checkPhoneUnique(SysUserBo user);
}
```

## 3. PO

1. **目的**：用于表示数据库中的实体，与表结构对应。
2. 特点：
   - 一般与数据库字段一一对应
   - 可被 ORM（如 MyBatis、JPA）直接映射
   - 一般不包含业务逻辑。
   - 常用于DAO层与数据库交互。

```java
// 对应数据库表 user
public class UserPO {
    private Long id;
    private String username;
    private String password;
    private LocalDateTime createTime;

    // getter/setter
}
```

> 如果你的项目确实需要区分持久化对象和领域对象，可以考虑命名如 `UserPO` 或者 `UserEntity`，但更常见的做法是直接用 `User` 作为实体类名，并通过注解等方式来标记它是持久化的实体。

## 4. DO

1. 目的：
   - **DO = Domain Object**，领域对象
   - 在**领域驱动设计（DDD）** 中使用，代表业务领域的核心模型，包含数据和行为。
   - 在非DDD的项目中，PO 与 DO 有时可混用

2. 特点：

   - 是“充血模型”（有方法+属性）。

   - 强调业务语义，如“下单”、“支付”等行为。

   - 与PO可能结构不同。

> 📌 DDD中，DO是核心，PO只是持久化载体。
>
> 通常直接命名为 `User`，因为它代表了业务领域的核心模型，包含了数据和相关的行为。

```java
// 领域对象：用户
public class UserDO {
    private Long id;
    private String username;
    private String password;
    private LocalDateTime createTime;

    // 业务行为
    public void changePassword(String oldPwd, String newPwd) {
        if (!password.equals(DigestUtils.md5Hex(oldPwd))) {
            throw new IllegalArgumentException("原密码错误");
        }
        this.password = DigestUtils.md5Hex(newPwd);
    }

    public boolean isPremiumUser() {
        return createTime.isBefore(LocalDateTime.now().minusYears(1));
    }
}
```

## 5. VO

1. **目的**：用于向前端展示数据，通常是页面展示的数据模型。

2. **特点**：

- 通常包含经过组合或格式化的展示数据
- 不一定与数据库字段一一对应
- 有时会包含前端所需的额外字段，如状态名、格式化日期等

```java
public class UserVO {
    private String username;
    private String email;
    private String createTimeFormatted; // 格式化后的时间字符串
}
```

# 2. 组合使用

如果你在项目中使用的是 Spring Boot + MyBatis 或 JPA 的架构，通常会看到如下组合使用：

| 层级             | 使用对象    | 职责说明                                  |
| ---------------- | ----------- | ----------------------------------------- |
| Controller       | `DTO`, `VO` | 接收请求参数（DTO）、返回响应（VO）       |
| Service          | `BO`, `DTO` | 业务逻辑处理，DTO → BO → DO，DO → BO → VO |
| DAO / Repository | `DO`        | 数据库操作，如增删改查                    |
| Entity (DB映射)  | `DO/PO`     | 与数据库表对应的实体                      |

## 1. 前端提交请求（JSON）

```json
POST /api/user/register
{
  "username": "jack",
  "email": "jack@example.com",
  "password": "123456"
}
```

## 2. Controller 层

```java
@PostMapping("/register")
public Result<UserVO> register(@RequestBody UserRegisterDTO dto) {
    UserVO vo = userService.register(dto);
    return Result.success(vo);
}
```

- 入参：`UserRegisterDTO`
- 出参：`UserVO`

## 3. Service 层

```java
public UserVO register(UserRegisterDTO dto) {
    // DTO → BO
    UserBO bo = new UserBO(dto.getUsername(), dto.getEmail(), dto.getPassword());

    // 校验逻辑
    if (!bo.isValidEmail()) {
        throw new BusinessException("邮箱不合法");
    }

    // BO → DO（保存）
    UserDO userDO = new UserDO();
    BeanUtils.copyProperties(bo, userDO);
    userDao.insert(userDO);

    // DO → VO（返回前端）
    UserVO vo = new UserVO();
    BeanUtils.copyProperties(userDO, vo);
    return vo;
}
```

## 4. DAO 层（MyBatis 示例）

```java
@Mapper
public interface UserDAO {
    void insert(UserDO user);
    UserDO findByUsername(String username);
}
```

## 5. UserDO 映射数据库表

```java
public class UserDO {
    private Long id;
    private String username;
    private String email;
    private String password;
    private LocalDateTime createTime;
}
```

数据库表结构示例：

```sql
CREATE TABLE user (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50),
    email VARCHAR(100),
    password VARCHAR(100),
    create_time DATETIME
);
```
