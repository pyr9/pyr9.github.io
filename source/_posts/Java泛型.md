---
title: Java泛型
date: 2022-11-09 22:31:12
tags: 
- 泛型
categories: Java基础
---

# 1.泛型概念

## 1.1 定义

泛型是JDK5中引入的特性，可以在编译阶段约束操作的数据类型，并进行检查

- java范型在生成字节码的时候会进行擦除
- 范型格式：`<数据类型>`
- 注意：泛型只能支持引用数据类型，因为定义的范型，其实在存入的时候还是会变成Object类型，而基本数据类型，不可以变成Object类型
- 指定范型类型后，可以传入该类类型及其子类类型

## 1.2 好处

- 统一数据类型
- 运行时期的问题，提到了编译期间，避免了强制类型转换出现的异常。

# 2 使用场景

## 2.1 范型类

### 2.1 定义

当编写一个类时，如果不确定类型，那么这个类就称为范型类,.如： ArrayList

- 创建这个类的时候，就可以确定类型
- 这里的类型可以定义为变量，是用来记录数据的类型的，可以写成T，E，K , V，也可以写成其他变量

### 2.2 格式

```
修饰符 class 类名<E>{

}
```

### 2.3 示例

```java
class MyArray<E> {
  Object[] obj = new Object[10];
  int size;

  // E表示不确定的类型，该类型在类后面定义过了
  public boolean add(E e) {
    obj[size] = e;
    size++;
    return true;
  }

  public E get(int index) {
    return (E) obj[index];
  }
}
```

```java
@Test
public void test() {
  MyArray<String> myArray = new MyArray<>();
  myArray.add("q");
  System.out.println(myArray.get(0));
}
```



## 2.2 范型方法

### 2.2.1 使用类型后面定义的类型

- 方法中形参类型不确定时，可以使用类型后面定义的类型。

- 所有的方法都可以使用这个类型

#### 示例

```java
class MyArray<E> {    
  public E get(int index) {
    return (E) obj[index];
   }
 }
```



### 2.2.2 **在方法上定义自己的类型**

只能在本方法中使用

#### 格式

```java
修饰符<E> 返回值类型 方法名(类型 变量名){

}
```

如：

```java
public<T> void show(T t){

}
```

#### 示例

```java
public <T> List<T> buildList(T... t) {
  return Arrays.asList(t);
}

public <T> T getValue(T t) {
  return t;
}
```

```java
@Test
public void test() {
  List<Integer> integers = buildList(1, 2, 3);
  System.out.println(integers);

  Integer value = getValue(1);
  System.out.println(value);

  String str = getValue("111");
  System.out.println(str);
}
```



## 2.3 范型接口

当编写一个接口时，如果不确定类型，那么这个类就称为范型接口，如： list

```java
修饰符 interface 接口名<E>{

}
```

如：

```java
修饰符 interface 接口名<E>{

}
```

### 1 使用一： 实现类给出具体类型

```java
public class MyList implements List<String> {
    @Override
    public boolean add(String s) {
        return false;
    }
}
```

```java
@Test
public void test() {
  MyList list = new MyList();
  list.add("string");
}
```



### 2 使用二：实现类继续延续泛型，创建对象时再确定

```java
public class MyList2<E> implements List<E> {
    @Override
    public boolean add(E e) {
        return false;
    }
}
```

```java
@Test
public void test() {
  MyList2<String> list2 = new MyList2();
  list2.add("string");
}
```

# 3 泛型通配符

- `?`可以表示不确定的类型

- `? extends E`: 表示只能传递E或者E的子类

- `? super E`: 表示只能传递E或者E的父类

## 3.1 示例

```java
class Father {
  public String id;
  public String name;
  public Father(String id, String name) {
    this.id = id;
    this.name = name;
  }
  public void study(){
    System.out.println("学习状态中------");
  }
}

class Son extends Father {
  public Son(String id, String name) {
    super(id, name);
  }
}

class GrandSon extends Son {
  public GrandSon(String id, String name) {
    super(id, name);
  }
}
```

- ? extends Son

```java
public void study(List<? extends Son> array) {
        array.forEach(Son::study);
}
```



<img src="https://tva1.sinaimg.cn/large/008vxvgGly1h80c7s1nt7j30p20cmgmq.jpg" style="zoom:50%;" />

- ? extends Son

```java
public void study(List<? super Son> array) {
  array.forEach(System.out::println);
}
```

<img src="https://tva1.sinaimg.cn/large/008vxvgGly1h80cw8fu3wj30p00bumy2.jpg" style="zoom:50%;" />



