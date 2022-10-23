---
title: Spring data JPA的CRUD
date: 2022-10-19 22:49:49
tags: 
- 数据库
- springData JPA
categories: JPA
---

# 1 Spring data JPA的CRUD

## 1.1 引入依赖

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

## 1.2 配置spring Data JPA

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

## 1.3 添加实体类

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

## 1.4 编写Repository

```java
package org.pyr.repositories;

import org.pyr.entity.Customer;
import org.springframework.data.repository.CrudRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface CustomerRepository extends CrudRepository<Customer, Long> {
}
```

## 1.4 编写测试类

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
