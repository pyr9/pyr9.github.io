---


title: Java8新特性之Stream流
date: 2021-10-04 22:36:47
tags: stream流
categories: Java基础

---

### Java8新特性之Stream流

### 1.什么是Stream？

Java 8 API添加了一个新的抽象称为流Stream，可以让你以一种声明的方式处理数据。通过声明性方式，能够对集合中的每个元素进行一系列并行或串行的流水线操作。例：

```java
public class StreamDemo1 {
    public static void main(String[] args) {
        int[] numbers = {1,2,3};
        int num2 = IntStream.of(numbers).sum();
        System.out.println(num2);
    }
}
```



### 2.创建流的几种方式

<!-- more -->

#### 2.1 使用集合

```java
ArrayList arrayList = new ArrayList();
arrayList.stream();
arrayList.parallelStream();  
```

#### 2.2 使用数组

```java
 Arrays.stream(new int[]{1,2,3});
```

#### 2.3 数字Stream

```java
IntStream.of(1,2,3);
IntStream.rangeClosed(1,10);
```

#### 2.4 使用Stream.generate()

```java
Stream.generate(()-> "stream").limit(5);
```



### 3. 常用方法

流模型的操作很丰富，这里介绍一些常用的API。这些方法可以被分成两种：

>- 延迟方法：返回值任然是Stream接口自身类型的方法，因此支持链式调用(除了终结方法外，其他都是方法均为延迟方法)
>- 终结方法：返回值类型不再是Stream接口自身类型的方法。

#### 3.1 延迟方法

##### 1. filter

可以通过`filter`方法将一个流转换成另一个子集流。

```java
Stream.of(1,2,3).filter(i -> i>2).forEach(System.out::println);
```

##### 2.peek

用于debug，foreach是最终操作

```
Stream.of("张三","李四","王麻子").peek(System.out::println).forEach(System.out::println);
```

##### 3.map

将流中的元素映射到另一个流中

```java
Stream.of("张三","李四","王麻子").map(String::length).forEach(System.out::println);
```

##### 4.limit

对流进行截取，只取用前n个

```java
Stream.of("张三","李四","王麻子").limit(2).forEach(System.out::println);
```

##### 5.skip

跳过前几个

```
Stream.of("张三","李四","王麻子").skip(2).forEach(System.out::println);
```

#### 3.2 终止方法

##### 1.forEach

迭代

```java
Stream.of(1,2,3).forEach(System.out::println);
```

##### 2.count

计算个数

```
System.out.println(Stream.of("张三","李四","王麻子").count());
```

##### 3.collect

收集到list

```java
List<String> list = Stream.of(str.split(" ")).collect(Collectors.toList());
System.out.println("收集到list:"+list);
```

##### 4.reduce

从Stream中生成一个值

- 使用reduce拼接字符串

```java
Optional<String> newStr = Stream.of(str.split("")).reduce((str1, str2) -> str1 + "-" + str2);
System.out.println(newStr.orElse(""));
```

- 使用reduce拼接字符串-+ 带有初始值

```java
String newStr2 = Stream.of(str.split("")).reduce("", (str1, str2) -> str1 + "-" + str2);
System.out.println(newStr2);
```

- 使用reduce计算所有单词总长度

```java
Optional<Integer> length = Stream.of(str.split(" ")).map(s -> s.length()).reduce((len1, len2) -> len1 + len2);
System.out.println("length:" + length.get());
```

##### 5.max

求最大值

```java
Optional<String> maxValue = Stream.of(str.split(" ")).max((str1, str2) -> str1.length() - str2.length());
System.out.println("maxValue:" + maxValue.get());
```

##### 6.findFirst

找第一个元素

```java
Optional<String> firstValue = Stream.of(str.split(" ")).findFirst();
System.out.println("firstValue:" + firstValue.get());
```

### 4. 惰性求值

惰性求值：终结没有调用的情况下，延迟方法不会执行。例：

```java
import java.util.stream.IntStream;

public class StreamDemo1 {
    public static void main(String[] args) {
        int[] numbers = {1, 2, 3};
        int num2 = IntStream.of(numbers).map(StreamDemo1::doubleNum).sum();
        System.out.println(num2);
      
      // 这里没有执行中间操作
        IntStream.of(numbers).map(StreamDemo1::doubleNum);
    }

    private static int doubleNum(int i) {
        System.out.println("执行了乘以2");
        return i * 2;
    }
}
```

```
> Task :StreamDemo1.main()
执行了乘以2
执行了乘以2
执行了乘以2
12
```

中间操作就是：返回stream的操作，如：map

终止操作就是：sum()

从输出结果可以看出doubleNum执行了3次

### 5.并行流

#### 5.1 创建一个并行流

```java
public class ParallelStream {
    public static void main(String[] args) {
        IntStream.range(1,100).parallel().peek(ParallelStream::debugger).count();
    }
    private static void debugger(int i) {
        System.out.println("debugger"+i);
    }
}
```

输出：

```
> Task :ParallelStream.main()
debugger90
debugger81
debugger82
debugger83
debugger84
debugger85
debugger86
debugger65
debugger66
debugger67
...
```

- 多次调用parallel / sequential，以最后一个为准，如：

```java
public class ParallelStream {
    public static void main(String[] args) {
        IntStream.range(1, 10)
                .parallel().peek(ParallelStream::debugger)
                .sequential().peek(ParallelStream::debugger2)
                .count();
    }

    private static void debugger2(int i) {
        System.err.println("debugger2: " + i);
    }

    private static void debugger(int i) {
        System.out.println("debugger1: " + i);
    }
}
```

#### 5.2 并行流使用自己定义的线程池

> 原因：避免使用默认线程池，防止任务被阻塞

```java
import java.util.concurrent.ForkJoinPool;
import java.util.stream.IntStream;

public class ParallelStream {
    public static void main(String[] args) {
        ForkJoinPool forkJoinPool = new ForkJoinPool(20);
        forkJoinPool.submit(() -> IntStream.range(1, 10)
                .parallel().peek(ParallelStream::debugger)
                .count());
        forkJoinPool.shutdown();

        synchronized (forkJoinPool) {
            try {
                forkJoinPool.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

    }

    private static void debugger(int i) {
        System.out.println(Thread.currentThread().getName() + "debugger1: " + i);
    }
}
```

### 6. Stream 流的运行机制

所有的操作都是链式调用，每个操作只会对每个元素操作一次；（是通过维护一个链表实现的。）具体实现：

> - 每个中间操作都会返回一个新的流，每个流里面都会有一个SourceStage属性，所有流的SourceStage属性都指向同一个地方head【就是原始流的头部】；
> - 如果一个中间操作之后还有中间操作，那么这个中间操作对应的流中nextStage属性就会执行下一个中间操作对应的流，否则就是null

注：parallel / sequential也是中间操作，但是他们呢不创建流，只是修改head 里的并行标志：parallel
