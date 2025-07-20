---
title: mybatis-plus字段自动填充
date: 2025-07-19 15:55:43
tags:
categories: Java基础
---

# 1. 主键ID生成

## 1.1. 在主键上加注解

```java
@Data
public class User {
    @TableId(type = IdType.ID_WORKER)
    private Long id;
}
```

## 1.2. 全局配置默认ID生成策略

```yml
mybatis-plus:
  global-config:
    db-config:
      id-type: auto # 可选值有：auto, input, assign_id, assign_uuid, none, id_worker等
```

## 1.3. 定义一个全局 ID 生成器 Bean

```java
import com.baomidou.mybatisplus.core.incrementer.IdentifierGenerator;
import com.baomidou.mybatisplus.core.incrementer.DefaultIdentifierGenerator;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MyBatisPlusConfig {
   /**
     * 使用网卡信息绑定雪花生成器
     * 防止集群雪花ID重复
     */
    
    @Bean
    public IdentifierGenerator idGenerator() {
        return new DefaultIdentifierGenerator(NetUtil.getLocalhost());
    }
}
```

# 2. 字段自动填充

## 2.1. entity加注解

```java
    /**
     * 创建者
     */
    @TableField(fill = FieldFill.INSERT)
    private Long createBy;

    /**
     * 创建时间
     */
    @TableField(fill = FieldFill.INSERT)
    private Date createTime;
```

## 2.2. 自定义实现类 InjectionMetaObjectHandler

```java
@Component
public class InjectionMetaObjectHandler implements MetaObjectHandler {

    /**
     * 插入填充方法，用于在插入数据时自动填充实体对象中的创建时间、更新时间、创建人、更新人等信息
     *
     * @param metaObject 元对象，用于获取原始对象并进行填充
     */
    @Override
    public void insertFill(MetaObject metaObject) {
        try {
            if (ObjectUtil.isNotNull(metaObject) && metaObject.getOriginalObject() instanceof BaseEntity baseEntity) {
                // 获取当前时间作为创建时间和更新时间，如果创建时间不为空，则使用创建时间，否则使用当前时间
                Date current = ObjectUtils.notNull(baseEntity.getCreateTime(), new Date());
                baseEntity.setCreateTime(current);
                baseEntity.setUpdateTime(current);

                // 如果创建人为空，则填充当前登录用户的信息
                if (ObjectUtil.isNull(baseEntity.getCreateBy())) {
                    LoginUser loginUser = getLoginUser();
                    if (ObjectUtil.isNotNull(loginUser)) {
                        Long userId = loginUser.getUserId();
                        // 填充创建人、更新人和创建部门信息
                        baseEntity.setCreateBy(userId);
                        baseEntity.setUpdateBy(userId);
                        baseEntity.setCreateDept(ObjectUtils.notNull(baseEntity.getCreateDept(), loginUser.getDeptId()));
                    }
                }
            } else {
                Date date = new Date();
                this.strictInsertFill(metaObject, "createTime", Date.class, date);
                this.strictInsertFill(metaObject, "updateTime", Date.class, date);
            }
        } catch (Exception e) {
            throw new ServiceException("自动注入异常 => " + e.getMessage(), HttpStatus.HTTP_UNAUTHORIZED);
        }
    }

    /**
     * 更新填充方法，用于在更新数据时自动填充实体对象中的更新时间和更新人信息
     *
     * @param metaObject 元对象，用于获取原始对象并进行填充
     */
    @Override
    public void updateFill(MetaObject metaObject) {
        try {
            if (ObjectUtil.isNotNull(metaObject) && metaObject.getOriginalObject() instanceof BaseEntity baseEntity) {
                // 获取当前时间作为更新时间，无论原始对象中的更新时间是否为空都填充
                Date current = new Date();
                baseEntity.setUpdateTime(current);

                // 获取当前登录用户的ID，并填充更新人信息
                Long userId = LoginHelper.getUserId();
                if (ObjectUtil.isNotNull(userId)) {
                    baseEntity.setUpdateBy(userId);
                }
            } else {
                this.strictUpdateFill(metaObject, "updateTime", Date.class, new Date());
            }
        } catch (Exception e) {
            throw new ServiceException("自动注入异常 => " + e.getMessage(), HttpStatus.HTTP_UNAUTHORIZED);
        }
    }
}
```
