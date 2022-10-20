---
title: JPA
date: 2022-10-11 23:04:31
tags:
categories: 
---

public interface JpaRepository<T, ID> extends PagingAndSortingRepository<T, ID>, QueryByExampleExecutor<T> 

`JpaRepository`接口= 基本`CRUD`功能+分页+按“实例”查询



**Query by Example**

**只支持字符串** start/contains/ends/regex 匹配和其他属性类型的精确匹 配。 

.将Repository继承QueryByExampleExecutor

```java
 @Test 
   public void test02(){ 
    Customer customer = new Customer(); 
     customer.setCustAddress("beijing"); 
     // 匹配器， 去设置更多条件匹配 
     ExampleMatcher matcher = ExampleMatcher.matching() 
       .withIgnoreCase("custAddress"); 
     Example<Customer> example = Example.of(customer,matcher);
     System.out.println(repository.findAll(example)); 
   }
```



**Querydsl** 

QueryDSL是基于ORM框架或SQL平台上的**一个通用查询框架**。借助QueryDSL可以在任何支持的ORM框架或SQL平台 **上以通用API方式构建查询。**

添加maven插件\>apt‐maven‐plugin ,这个插件是为了让程序自动生成query type(查询实体，命名方式为："Q"+对应实体名)。 

```java
14 public void test02(){ 
  QCustomer qCustomer = QCustomer.customer; 
  Iterable<Customer> all = repository.findAll(qCustomer.id.in(1L, 5L).and(qCustomer.firstName.in("徐庶", "王五"))); 17 System.out.println(all);
```

