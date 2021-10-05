---
title: Java8的新特性之lambda表达式
date: 2021-10-04 22:32:27
tags: lambda表达式
categories: java8新特性
---

### 1.函数接口

函数接口（`@FunctionalInterface`）需要满足两个条件：

- 类型是接口
- 有且只有一个抽象方法

例如：Runnable接口

```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```

其实这也是要求我们接口的设计尽量小，符合单一责任制，一个接口只做一个事，这样使用lambda就会比较方便。

### 2. lambda表达式是个啥？

**lambda表达式**是**Java8**的新特性，它就是就是一个匿名函数，箭头左边是函数的参数，右边是函数的执行体。

<!-- more -->

```java
public class Test {
    public static void main(String[] args) {
      new Thread(new Runnable() {
          @Override
          public void run() {
              System.out.println("正常创建一个线程");
          }
      }).start();
      # 实际上是返回了一个实现了Runnable的实例，箭头左边是参数，右边是方法体
      new Thread(()-> System.out.println("Lambda创建一个线程")).start();
    }
}
```

以上面的代码为例，Runnable接口仅有一个run方法，并且该方法没有参数，所以编译器可以自动推断出箭头后的内容为run方法的方法体。

如果Runnable接口中含有多个方法，编译器将无法编译lambda表达式，可以看出，lambda表达式是根据编译器的隐式推断来简化代码的。所以，**lambda表达式需要函数式接口的支持。**

**实例:**

```java
interface MoneyFormat{
    String format(int i);
}
public class MoneyDemo {

    public final int money;

    public MoneyDemo(int money) {
        this.money = money;
    }

    public void printMoney(MoneyFormat moneyFormat){
        System.out.println(String.format("金额数为：" + moneyFormat.format(this.money)));
    }

    public static void main(String[] args) {
        MoneyDemo moneyDemo = new MoneyDemo(999);
        moneyDemo.printMoney(i -> new DecimalFormat("#,##").format(i));
    }
}
```

由上可见，我们不关心接口的名字是什么，只关心输入是int，输出 是string，因此上面代码可优化为：

删除interface MoneyFormat，使用jdk8带的函数接口Function<Integer,String>替代

```java
public class MoneyDemo {

    public final int money;

    public MoneyDemo(int money) {
        this.money = money;
    }

    public void printMoney(Function<Integer,String> moneyFormat){
        System.out.println(String.format("金额数为：" + moneyFormat.apply(this.money)));
    }

    public static void main(String[] args) {
        MoneyDemo moneyDemo = new MoneyDemo(999);
        moneyDemo.printMoney(i -> new DecimalFormat("#,##").format(i));
    }
}
```

### 

## 3. lambda表达式常用的函数式接口

主要是分布在java.util.function包中，下面只简单列举2种：

#### 2.1 Supplier接口

- `java.util.function.Supplier<T>` 接口仅含有一个无参方法,`T get()`
- `Supplier<T>` 接口是生产型接口,接口泛型指定什么类型,就返回什么泛型

```java
import java.util.function.Supplier;

public class SupplierDemo {
    public String supplyName(Supplier<String> supplier) {
        return supplier.get();
    }
  
    public static void main(String[] args) {
        SupplierDemo supplierDemo = new SupplierDemo();
        String name = supplierDemo.supplyName(() -> "张三");
        System.out.println(name);
    }
}
```



### 3.2 Consumer接口

- `java.util.function.Consumer接口<T>` 接口仅含有一个有参方法,`void accept(T t)`
- `Consumer接口<T>` 接口是消费型接口,接口泛型制定什么类型,就接受什么泛型

```java
import java.util.function.Consumer;

public class ConsumerDemo {

    public void printName(String name, Consumer<String> consumer) {
        consumer.accept(name);
    }

    public static void main(String[] args) {
        ConsumerDemo consumerDemo = new ConsumerDemo();
        consumerDemo.printName("张三", (String name) -> System.out.println("姓名为 : " + name));
    }
}
```



### 

