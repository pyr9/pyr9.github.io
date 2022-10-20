---
title: 基于Hibernate数据库操作
date: 2022-10-16 22:44:03
tags: 
- Hibernate
- 数据库
categories: JPA
---

# 1 基于Hibernate数据库操作

## 1.1 操作步骤

### 1.1.1 导入依赖

```java
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
```

### 1.1.2 增加实体类

```java
package tt.pyr.entity;

import javax.persistence.*;

@Entity
@Table(name = "cst_customer")
public class Customer {

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
}
```

### 1.1.3 配置**hibernate.cfg.xml** 

```java
<?xml version = "1.0" encoding = "utf-8"?>
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">
<hibernate-configuration>
    <session-factory>
        <property name="hibernate.connection.url">jdbc:mysql://localhost:3306/springdata_jpa</property>
        <property name="hibernate.connection.username">root</property>
        <property name="hibernate.connection.password">19980617pyr</property>
        <property name="hibernate.connection.driver_class">com.mysql.cj.jdbc.Driver</property>
        <property name="hibernate.show_sql">true</property>
        <property name="format_sql">true</property>
        <property name="hbm2ddl.auto">update</property>
        <mapping class="tt.pyr.entity.Customer"></mapping>
    </session-factory>
</hibernate-configuration>

```

### 1.1.4 编写测试用例

```java
package test;

import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.Transaction;
import org.hibernate.boot.MetadataSources;
import org.hibernate.boot.registry.StandardServiceRegistry;
import org.hibernate.boot.registry.StandardServiceRegistryBuilder;
import org.junit.Before;
import org.junit.Test;
import tt.pyr.entity.Customer;

public class HibernateTest {
    private SessionFactory sf;

    @Before
    public void init() {
        StandardServiceRegistry registry = new StandardServiceRegistryBuilder().configure("/hibernate.cfg.xml").build();
        //2. 根据服务注册类创建一个元数据资源集，同时构建元数据并生成应用一般唯一的的session工厂
        sf = new MetadataSources(registry).buildMetadata().buildSessionFactory();
    }

    @Test
    public void testInsert() {
        Session sess = sf.openSession();
        Transaction tx = sess.beginTransaction();
        Customer customer = new Customer();
        customer.setCustName("张三");
        sess.save(customer);
        tx.commit();
        sess.close();
        sf.close();
    }

    @Test
    public void testSaveOrUpdate() {
        Session sess = sf.openSession();
        Transaction tx = sess.beginTransaction();
        Customer customer = new Customer();
        customer.setCustName("里斯");
        sess.saveOrUpdate(customer);
        tx.commit();
        sess.close();
        sf.close();
    }

    @Test
    public void testRemove() {
        Session sess = sf.openSession();
        Transaction tx = sess.beginTransaction();
        Customer customer = new Customer();
        customer.setCustId(1L);
        sess.remove(customer);
        tx.commit();
        sess.close();
        sf.close();
    }
  
      @Test
    public void testHQL() {
        Session sess = sf.openSession();
        Transaction tx = sess.beginTransaction();

        String sql = "Update Customer set custName=:custName where custId=:id";
        sess.createQuery(sql).setParameter("custName", "徐庶").setParameter("id", 2L).executeUpdate();
        tx.commit();
        sess.close();
        sf.close();
    }
}
```

## 1.2 HQL

### 1.2.1 是什么？

- HQL是Hibernate Query Language（Hibernate 查询语言）的缩写，提供更加丰富灵活、更为强大的查询能力；HQL更接近SQL语句查询语法。
- Hibernate查询语言(HQL)与SQL(结构化查询语言)相同，但不依赖于数据库表。 我们在HQL中使用类名，而不是表名,它是数据库独立的查询语言。

### 1.1.2 HQL查询的步骤

- 获得Hibernate Session对象
- 编写HQL语句
- 调用Session的createQuery方法创建查询对象
- 如果HQL语句包含参数，则调用Query的setXxx方法为参数赋值
- 调用Query对象的list等方法返回查询结果。

```java
package test;

import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.Transaction;
import org.hibernate.boot.MetadataSources;
import org.hibernate.boot.registry.StandardServiceRegistry;
import org.hibernate.boot.registry.StandardServiceRegistryBuilder;
import org.junit.Before;
import org.junit.Test;
import tt.pyr.entity.Customer;

public class HibernateTest {
    private SessionFactory sf;

    @Before
    public void init() {
        StandardServiceRegistry registry = new StandardServiceRegistryBuilder().configure("/hibernate.cfg.xml").build();
        sf = new MetadataSources(registry).buildMetadata().buildSessionFactory();
    }
  
      @Test
    public void testHQL() {
        Session sess = sf.openSession();
        Transaction tx = sess.beginTransaction();

        String sql = "Update Customer set custName=:custName where custId=:id";
        sess.createQuery(sql).setParameter("custName", "徐庶").setParameter("id", 2L).executeUpdate();
        tx.commit();
        sess.close();
        sf.close();
    }
}
```

