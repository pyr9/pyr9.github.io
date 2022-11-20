---
title: springboot整合graphql
date: 2021-10-06 17:49:08
tags: graphql
categories: Springboot
---

 GraphQL是比REST更高效、强大和灵活的**新一代API标准**。详细的可以看官网[GraphQL](https://graphql.cn/)。

下面介绍一个Spring boot整合graphql简单的例子。

<!-- more -->

- 项目准备：相关依赖的引入：

  ```properties
  <dependencies>
  		<!--web -->
  		<dependency>
  			<groupId>org.springframework.boot</groupId>
  			<artifactId>spring-boot-starter-web</artifactId>
  		</dependency>
  		<!--lombok -->
  		<dependency>
  			<groupId>org.projectlombok</groupId>
  			<artifactId>lombok</artifactId>
  			<optional>true</optional>
  		</dependency>
  		<!--graphql -->
  		<dependency>
  			<groupId>com.graphql-java-kickstart</groupId>
  			<artifactId>graphql-spring-boot-starter</artifactId>
  			<version>11.0.0</version>
  		</dependency>
  		<!--playground -->
  		<dependency>
  			<groupId>com.graphql-java-kickstart</groupId>
  			<artifactId>playground-spring-boot-starter</artifactId>
  			<version>11.0.0</version>
  		</dependency>
  </dependencies>
  ```

#### Query

- 定义schema，分别为：

  `query.graphqls`

  ```java
  type Query{
    billingAccount(id: ID): BillingAccount
  }
  ```

  `billingAccount.graphqls`

  ```java
  type BillingAccount {
    id: ID!
    name: String!
    currency: Currency
  }
  ```

  `currency.graphqls`

  ```java
  enum Currency{
      RMB,
      USD
  }
  ```

- 定义基础要操作的模型

  ```java
  @Builder
  @Value
  public class BillingAccount {
      UUID id;
      String name;
      Currency currency;
  }
  ```

  ```java
  public enum Currency {
      RMB, USD
  }
  ```

- 定义resolver

  ```java
  @Slf4j
  @Component
  public class BillingAccountResolver implements GraphQLQueryResolver {
      public BillingAccount billingAccount(UUID id){
          log.info("receive billingAccount id is: "+ id);
          return BillingAccount.builder()
                  .id(id)
                  .currency(Currency.RMB)
                  .name("张三")
                  .build();
      }
  }
  ```



- 效果图

![img](/images/resolver.png)

#### mutation

- 定义schema

```
type Mutation{
    createBillingAccount(input: CreateBillingAccountInput): BillingAccount
}
```

- 定义input

```
input CreateBillingAccountInput{
    name: String
}
```

- 定义input 对应的model

```java
@Data
public class CreateBillingAccountInput {
  String name;
}
```

- 定义mutationResolver

```java
@Component
public class BillingAccountMutation implements GraphQLMutationResolver {

  public BillingAccount createBillingAccount(CreateBillingAccountInput input){

    return BillingAccount.builder()
      .id(UUID.randomUUID())
      .currency(Currency.RMB)
      .name(input.getName())
      .build();
  }
}
```



效果图

!![mutation](/images/mutation.png)

