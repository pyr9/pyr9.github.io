---
title: Spring Data JPA 使用JPQL和SQL
date: 2022-10-20 21:58:30
tags: 
- 数据库
- springData JPA
categories: JPA
---

# 2 spring Data JPA 使用JPQL

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

# 3  spring Data JPA 使用SQL

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

