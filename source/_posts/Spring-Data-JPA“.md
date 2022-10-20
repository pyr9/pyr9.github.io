---
title: Spring Data JPA“
date: 2022-10-19 22:49:49
tags: 
- 数据库
categories: JPA
---

# 1 **Spring data JPA**是什么？

- spirng data jpa是spring提供的一套**简化JPA开发的框架**，按照约定好的规则进行【方法命名】去写dao层接口，就可以在不写接口实现的情况下，实现对数据库的访问和操作。同时提供了很多除了CRUD之外的功能，如分页、排序、复杂查询等等

- **Spring Data JPA 旨在改** **进数据访问层的实现以提升开发效率**
- Spring Data JPA 让我们解脱了DAO层的操作，基本上所有CRUD都可以依赖于它来实现,在实际的工作工程中，推荐使用Spring Data JPA + ORM（如：hibernate）完成操作，这样在切换不同的ORM框架时提供了极大的方便，同时也使数据库层操作更加简单，方便解耦

# 2 整合步骤

## 2.1 引入依赖

```xml
dependencies>
        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-jpa</artifactId>
        </dependency>
        <!-- https://mvnrepository.com/artifact/com.alibaba/druid -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.2.8</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>5.3.19</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.data</groupId>
                <artifactId>spring-data-bom</artifactId>
                <version>2021.1.0</version>
                <scope>import</scope>
                <type>pom</type>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

## 2.2 配置spring Data JPA

```java
package org.pyr.config;

import com.alibaba.druid.pool.DruidDataSource;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
import org.springframework.orm.jpa.JpaTransactionManager;
import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
import org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.annotation.EnableTransactionManagement;

import javax.persistence.EntityManagerFactory;
import javax.sql.DataSource;

@Configuration
@EnableJpaRepositories("org.pyr.repositories")
@EnableTransactionManagement
public class JPAConfig {

  @Bean
  public DataSource dataSource() {
    DruidDataSource dataSource = new DruidDataSource();
    dataSource.setUsername("root");
    dataSource.setPassword("19980617pyr");
    dataSource.setDriverClassName("com.mysql.jdbc.Driver");
    dataSource.setUrl("jdbc:mysql://localhost:3306/springdata_jpa");
    return dataSource;
  }

  @Bean
  public LocalContainerEntityManagerFactoryBean entityManagerFactory() {

    HibernateJpaVendorAdapter vendorAdapter = new HibernateJpaVendorAdapter();
    vendorAdapter.setGenerateDdl(true);

    LocalContainerEntityManagerFactoryBean factory = new LocalContainerEntityManagerFactoryBean();
    factory.setJpaVendorAdapter(vendorAdapter);
    factory.setPackagesToScan("org.pyr.entity");
    factory.setDataSource(dataSource());
    return factory;
  }

  @Bean
  public PlatformTransactionManager transactionManager(EntityManagerFactory entityManagerFactory) {

    JpaTransactionManager txManager = new JpaTransactionManager();
    txManager.setEntityManagerFactory(entityManagerFactory);
    return txManager;
  }
}
```

## 2.3 添加实体类

```java
package org.pyr.entity;

import javax.persistence.*;

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

## 2.4 编写测试类

```java
package org.pyr;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.pyr.config.JPAConfig;
import org.pyr.entity.Customer;
import org.pyr.repositories.CustomerRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import java.util.Optional;

@ContextConfiguration(classes = JPAConfig.class)
@RunWith(SpringJUnit4ClassRunner.class)
public class JPATest {
    @Autowired
    CustomerRepository customerRepository;

    @Test
    public void testFind(){
        Optional<Customer> customer = customerRepository.findById(3L);
        System.out.println(customer.get());
    }

    @Test
    public void testSave(){
        Customer customer = new Customer();
        customer.setCustName("aa");
        customerRepository.save(customer);
    }

    @Test
    public void testUpdate(){
        Customer customer = new Customer();
        customer.setCustId(3L);
        customer.setCustName("aa123");
        customerRepository.save(customer);
    }

    @Test
    public void testDelete(){
        Customer customer = new Customer();
        customer.setCustId(3L);
        customer.setCustName("aa123");
        customerRepository.delete(customer);
    }
}
```

# 3 spring Data JPA 使用JPQL

JPQL（Java Presistence Query Language ）是EJB3.0中的JPA造出来的对象查询语言。JPQL是完全面向对象的，具备继承、多态和关联等特性，和hibernate HQL很相似。

- Repository 方法

```java
package org.pyr.repositories;

import org.pyr.entity.Customer;
import org.springframework.data.jpa.repository.Modifying;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.PagingAndSortingRepository;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;
import org.springframework.transaction.annotation.Transactional;

@Repository
public interface CustomerJPQLRepository extends PagingAndSortingRepository<Customer, Long> {

    @Query("From Customer where custName = :custName")
    Customer findCustomerByCustName(@Param("custName") String custName);

    @Query("From Customer where custName = ?1")
    Customer findCustomerByCustName2(String custName);

    @Transactional // 通常在业务逻辑中service增加，而不是这里
    @Modifying
    @Query("UPDATE Customer set custName = :custName where custId = :custId")
    int updateCutomserNameById(@Param("custName") String custName, @Param("custId") long custId);

    @Transactional // 通常在业务逻辑中service增加，而不是这里
    @Modifying
    @Query("DELETE from Customer where custId = :custId")
    int deleteCutomserById(@Param("custId") long custId);
}

```

- 测试方法

```java
package org.pyr;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.pyr.config.JPAConfig;
import org.pyr.entity.Customer;
import org.pyr.repositories.CustomerJPQLRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@ContextConfiguration(classes = JPAConfig.class)
@RunWith(SpringJUnit4ClassRunner.class)
public class JPATest2 {
    @Autowired
    CustomerJPQLRepository customerJPQLRepository;

    @Test
    public void testFindByName(){
        Customer customer = customerJPQLRepository.findCustomerByCustName("徐庶143");
        System.out.println(customer);
    }

    @Test
    public void testFindById(){
        Customer customer = customerJPQLRepository.findCustomerByCustName2("徐庶143");
        System.out.println(customer);
    }

    @Test
    public void testUpdate(){
        customerJPQLRepository.updateCutomserNameById("lisa", 2L);
    }

    @Test
    public void testDelete(){
        customerJPQLRepository.deleteCutomserById(2L);
    }
}
```

# 4  spring Data JPA 使用SQL

- Repository 方法

```java
    @Query(value = "select * from cst_customer where cust_name = :custName", nativeQuery = true)
    Customer findCustomerByCustNameWithNative(@Param("custName") String custName);
```

- 测试

```java
    @Test
    public void testFindCustomerByCustNameWithNative(){
        Customer customer = customerJPQLRepository.findCustomerByCustNameWithNative("aa");
        System.out.println(customer);
    }
```

