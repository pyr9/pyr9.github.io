---
title: 基于JPA的数据库操作
date: 2022-10-19 21:55:20
tags:
- 数据库
categories: JPA
---

产生原因：**如果单独使用hibernate的API来进行持久化操作，则不能随意切换其他ORM框架**

# 1 操作步骤

## 1.1 导入依赖

```xml
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <version>8.0.29</version>
  <scope>test</scope>
</dependency>
<dependency>
  <groupId>org.hibernate</groupId>
  <artifactId>hibernate-core</artifactId>
  <version>5.6.8.Final</version>
</dependency>
<dependency>
  <groupId>junit</groupId>
  <artifactId>junit</artifactId>
  <version>4.13</version>
  <scope>test</scope>
</dependency>
```

## 1.2 添加META-INF\persistence.xml 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="http://java.sun.com/xml/ns/persistence" version="2.0">
    <!--需要配置persistence‐unit节点持久化单元：name：持久化单元名称
    transaction‐type：事务管理的方式 JTA：分布式事务管理 RESOURCE_LOCAL：本地事务管理-->
    <persistence-unit name="hibernateJPA" transaction-type="RESOURCE_LOCAL">
        <!--        jpa的实现方式 -->
        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
        <properties>
            <!--            配置jpa实现方的配置信息-->
            <property name="javax.persistence.jdbc.url" value="jdbc:mysql://localhost:3306/springdata_jpa"/>
            <property name="javax.persistence.jdbc.user" value="root"/>
            <property name="javax.persistence.jdbc.password" value="19980617pyr"/>
            <property name="javax.persistence.jdbc.driver" value="com.mysql.cj.jdbc.Driver"/>
            <!--            配置jpa实现方(hibernate)的配置信息-->
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
            <property name="hibernate.hbm2ddl.auto" value="update"/>
            <property name="hibernate.dialect" value="org.hibernate.dialect.MySQL5InnoDBDialect" />
        </properties>
    </persistence-unit>
</persistence>
```

## 1.3 增加测试实体类

```java
package entity;

import javax.persistence.*;
import java.io.Serializable;

@Entity
@Table(name = "cst_customer")
public class Customer{

    @Id //声明主键的配置
    @GeneratedValue(strategy = GenerationType.IDENTITY) //:配置主键的生成策略: .IDENTITY ：自增，mysql
    @Column(name = "cust_id") // 配置属性和字段的映射关系  name：数据库表中字段的名称
    private Long custId; //客户的主键

    @Column(name = "cust_name")
    private String custName;//客户名称

    public void setCustId(Long custId) {
        this.custId = custId;
    }

    public void setCustName(String custName) {
        this.custName = custName;
    }

    @Override
    public String toString() {
        return "Customer{" +
                "custId=" + custId +
                ", custName='" + custName + '\'' +
                '}';
    }
}

```

## 1.4 测试

**准备工作**

```java
public class Test {
    EntityManager entityManager;

    @Before
    public void init() {
        EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("hibernateJPA");
        entityManager = entityManagerFactory.createEntityManager();
    }
}
```

### Save

```java
 @org.junit.Test
    public void testSave() {
        EntityTransaction transaction = entityManager.getTransaction();
        transaction.begin();

        Customer customer = new Customer();
        customer.setCustName("张三");
        entityManager.persist(customer);

        transaction.commit();
        entityManager.close();
    }
```

### Delete

```java
    @org.junit.Test
    public void testRemove() {
        EntityTransaction transaction = entityManager.getTransaction();
        transaction.begin();
//        Customer customer =new Customer();
//        customer.setCustId(3L);
        Customer customer = entityManager.find(Customer.class, 3L);
        entityManager.remove(customer);
        transaction.commit();
        entityManager.close();
    }
```

⚠️：这里要删除的对象需要是数据库里查询到的，和entityManager发生过关联的对象，即持久状态。或者使用HQL。new出来的对象无法完成删除逻辑。

### Update

```java
    @org.junit.Test
    public void testUpdate() {
        EntityTransaction transaction = entityManager.getTransaction();
        transaction.begin();

        Customer customer = new Customer();
        customer.setCustName("张三123");
        customer.setCustId(3L);
        // 指定id即更新，没有id就新增
        Customer rs = entityManager.merge(customer);
        System.out.println(rs);
        transaction.commit();
        entityManager.close();
    }
```

- HQLUpdate

```java
    @org.junit.Test
    public void testHQLUpdate() {
        EntityTransaction transaction = entityManager.getTransaction();
        transaction.begin();

        String sqlQuery = "update Customer set custName=:name where custId=:id";
        entityManager.createQuery(sqlQuery)
                .setParameter("name", "徐庶143")
                .setParameter("id", 2L)
                .executeUpdate();

        transaction.commit();
        entityManager.close();
    }
```

- Sql查询

```java
    @org.junit.Test
    public void testSQLUpdate() {
        EntityTransaction transaction = entityManager.getTransaction();
        transaction.begin();

        String sqlQuery = "update cst_customer set cust_id=:name where cust_name=:id";
        entityManager.createNativeQuery(sqlQuery)
                .setParameter("name", "徐庶123")
                .setParameter("id", 2L)
                .executeUpdate();

        transaction.commit();
        entityManager.close();
    }
```

⚠️：这里使用createNativeQuery，表明和字段名都换成小写

### Select

- 立即查询

```java
    @org.junit.Test
    public void testFind() {
        EntityTransaction transaction = entityManager.getTransaction();
        transaction.begin();

        Customer rs = entityManager.find(Customer.class, 3L);
        System.out.println("========================");
        System.out.println(rs);
        transaction.commit();
        entityManager.close();
    }
```

- 懒加载查询

```java
    @org.junit.Test
    public void testLazyFind() {
        EntityTransaction transaction = entityManager.getTransaction();
        transaction.begin();

        Customer rs = entityManager.getReference(Customer.class, 3L);
        System.out.println("========================");
        System.out.println(rs);
        transaction.commit();
        entityManager.close();
    }
```

- HQL查询

```java
    @org.junit.Test
    public void testHQLSelect() {
        EntityTransaction transaction = entityManager.getTransaction();
        transaction.begin();

        String sqlQuery = "select c from Customer c";
        Query query = entityManager.createQuery(sqlQuery);
        List<Customer> resultList = query.getResultList();
        resultList.forEach(System.out::println);

        transaction.commit();
        entityManager.close();
    }
```



