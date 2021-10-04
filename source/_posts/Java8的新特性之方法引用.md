---
title: Java8的新特性之方法引用
date: 2021-10-04 22:30:22
tags:
  - 方法引用
categories:  jdk
---

### 1.方法引用是什么？

> Java8的新特性之二：方法引用。方法引用其实也离不开Lambda表达式。

- 方法引用通过方法的名字来指向一个方法。

- 方法引用可以使语言的构造更紧凑简洁，减少冗余代码。

- 方法引用使用一对冒号 **::** 。

## 2、方法引用的分类

下面，我们在 Car 类中定义了 4 个方法作为例子来区分 Java 中 4 种不同方法的引用。

<!-- more -->

|     类型     | 语法                      | 对应lambda表达式                          |
| :----------: | ------------------------- | ----------------------------------------- |
| 静态方法引用 | 类名::staticMethod        | (args) -> 类名.staticMethod(args)         |
| 实例方法引用 | instance::instance_Method | (args) -> instance.instance_Method(args)  |
| 对象方法引用 | 类名::instance_Method     | (inst,args) -> 类名.instance_Method(args) |
| 构建方法引用 | 类名::new                 | (args) -> new 类名(args)                  |



###  3、方法引用举例

#### 3.1 静态方法引用

- 实例1:

  ```java
      public static void main(String[] args) {
        //Consumer<String> consumer = s -> System.out.println(s); 输出的参数和输入的参数一致，可以缩写
          Consumer<String> consumer = System.out::println;
          consumer.accept("接受的数据");
      }
  ```

   实例中我们将 System.out::println 方法作为静态方法来引用。

- 实例2:

  ```java
  public class MethodReferenceDemo {
      static class Dog {
          private String name = "啸天犬";
  
          public static void bark(Dog dog) {
              System.out.println(dog + "狗叫了");
          }
  
          @Override
          public String toString() {
              return this.name;
          }
      }
       public static void main(String[] args) {
         //静态方法
          Dog dog = new Dog();
          Consumer<Dog> consumer2 = Dog::bark;
          consumer2.accept(dog);
      }
  }
  ```

#### 3.2 实例方法引用

```java
import java.util.function.Function;

public class MethodReferenceDemo {
    static class Dog {
        private int food_weight = 10;
        
        public int eat(int weight) {
            System.out.println("小狗吃了" + weight + "斤狗粮");
            this.food_weight -= weight;
            return this.food_weight;
        }
    }

    public static void main(String[] args) {
        Dog dog = new Dog();
        // Function<Integer, Integer> function = dog::eat;
        // UnaryOperator<Integer> function = dog::eat;
        // int weight = function.apply(3);
        IntUnaryOperator function = dog::eat;
        int weight = function.applyAsInt(3);
        System.out.println("还剩余" + weight + "斤狗粮");
    }
}
```

上面的eat方法，页可改写为`public int eat(Dog this, int weight) `编译也不会报错。因此=》

> ```
> JDk会默认把当前实例传入到非静态方法，参数名为this，位置是第一个
> ```

#### 3.3 构建方法引用

- 无参数的构造方法引用

```java
import java.util.function.Supplier;

public class MethodReferenceDemo {
    static class Dog {
        private String name = "啸天犬";

        @Override
        public String toString() {
            return this.name;
        }
    }

    public static void main(String[] args) {
        Supplier<Dog> supplier = Dog::new;
        System.out.println(supplier.get());
    }
}
```

- 有参数的构造方法的引用

```java
import java.util.function.Function;

public class MethodReferenceDemo {
    static class Dog {
        private String name = "啸天犬";

        public Dog(String name) {
            this.name = name;
        }
        @Override
        public String toString() {
            return this.name;
        }
    }

    public static void main(String[] args) {
        Function<String,Dog> function2 =  Dog::new;;
        System.out.println(function2.apply("小杂毛"));
    }
}
```

