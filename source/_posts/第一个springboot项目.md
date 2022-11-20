---
title: 第一个SpringBoot项目
date: 2021-12-25 20:13:48
tags: springboot
categories: Springboot
---

# 编写代码

- 进入官网 [Spring Initializr](https://start.spring.io/) 初始化项目
- 编写控制HelloController

```java
package com.pyr.spring.cloud.weather.controller;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class helloController {

  @RequestMapping("/hello")
  public String hello() {
    return "hello World!";
  }
}
```



# 编写测试用例

```java
package com.pyr.spring.cloud.weather.controller;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;

import static org.hamcrest.Matchers.equalTo;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureMockMvc
class helloControllerTest {

  @Autowired
  private MockMvc mockMvc;

  @Test
  void hello() throws Exception {
    mockMvc.perform(MockMvcRequestBuilders.get("/hello").accept(MediaType.APPLICATION_JSON))
      .andExpect(status().isOk())
      .andExpect(content().string(equalTo("hello World!")));
  }
}
```



# 启动项目

- idea 里面右击项目
- build/libs下运行 `Java -jar xxx`
- 项目目录下运行：`gradle bootRun`

![](https://tva1.sinaimg.cn/large/008i3skNly1gxr8m9n86ij30sq08ujrx.jpg)
