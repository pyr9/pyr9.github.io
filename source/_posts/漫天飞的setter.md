---
title: 漫天飞的setter
date: 2022-04-01 22:44:00
tags: 代码坏味道
categories: 代码之丑
---

# 漫天飞的setter

> - **重构手法**：
>   - 方法封装在对应的模型中  
>   - 使用构造函数，移除设值函数。
>   - 编写不变类

### 案例1

```java
public void approve(final long bookId) {
  ...
  book.setReviewStatus(ReviewStatus.APPROVED);
  ...
}
```

- 问题：使用到了setter, setter往往是缺乏封装的一种做法。你不知道数据会在何时被哪里修改，造成结果是别人的改动可能会使你的代码崩溃，同时会有并发问题
- 修正：使用函数封装了setter方法

```java
public void approve(final long bookId) {
  ...
  book.approve();
  ...
} 

class Book {
  public void approve() {
    this.reviewStatus = ReviewStatus.APPROVED;
  }
}
```

**进一步重构**

> 重构手法：编写不变类

改造之前的方法

```java
class Book {
  public Book approve() {
    return new Book(..., ReviewStatus.APPROVED, ...);
  }
}
```



### 案例2

```java
Book book = new Book();
book.setBookId(bookId);
book.setTitle(title);
book.setIntroduction(introduction);
```

- 问题：这种初始化的代码，压根没必要以setter的方式存在

- 修正：

  ```java
  Book book = new Book(bookId, title, introduction);
  ```



<img src="https://tva1.sinaimg.cn/large/e6c9d24ely1h0um32zjp7j20u01560x5.jpg" style="zoom:50%;" />
