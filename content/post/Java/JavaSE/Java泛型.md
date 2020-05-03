---
title: "Java泛型"
date: 2019-03-04T15:18:00+08:00
categories:
- Java
tags:
- Java
- 泛型
keywords:
- Java
- 泛型
---

JDK 5.0 中增加的泛型类型，是 Java 语言中类型安全的一次重要改进。泛型允许在定义类和接口的时候使用类型参数（type parameter）。声明的类型参数在使用时用具体的类型来替换。

<!--more-->

## Java泛型由来的动机

理解Java泛型最简单的方法是把它看成一种便捷语法，能节省你某些Java类型转换(casting)，例如：

```java
List<Apple> box = ...;
Apple apple = box.get(0);
```

box是一个装有Apple对象的List。get方法返回一个Apple对象实例，这个过程不需要进行类型转换。没有泛型，上面的代码需要写成这样：

```java
List box = ...;
Apple apple = (Apple) box.get(0);
```

泛型的主要好处就是让编译器保留参数的类型信息，执行类型检查，执行类型转换操作：编译器保证了这些类型转换的绝对无误。

## 泛型的构成

由泛型的构成引出了一个**类型变量**的概念。根据Java语言规范，类型变量是一种没有限制的标志符，因此产生于以下几种情况：

* 泛型类声明
* 泛型接口声明
* 泛型方法声明
* 泛型构造器(constructor)声明

### 泛型类和接口

如果一个类或接口上有一个或多个类型变量，那它就是泛型。类型变量由尖括号界定，放在类或接口名的后面。

Demo：

```java
public interface List<T> extends Collection<T> {
	
}
```

类型变量就如同一个参数，它提供给编译器用来类型检查的信息。

###　泛型方法和构造器(Constructor)

如果方法和构造器上声明了一个或多个类型变量，它们也可以泛型化。

Demo：

```java
public static <T> T getFirst(List<T> list){
	
}
```

这个方法将会接受一个List<T>类型的参数，返回一个T类型的对象。

**泛型类型在逻辑上看以看成是多个不同的类型，实际上都是相同的基本类型。**

## 关于子类型 

在Java中，跟其它具有面向对象类型的语言一样，类型的层级可以被设计成这样：

![Java_generics_1](/images/Java/Java_generics_1.png)

在Java中，类型T的子类型既可以是类型T的一个扩展，也可以是类型T的一个直接或非直接实现(如果T是一个接口的话)。因为“成为某类型的子类型”是一个具有传递性质的关系，如果类型A是B的一个子类型，B是C的子类型，那么A也是C的子类型。所有Java类型都是Object类型的子类型。

B类型的任何一个子类型A都可以被赋给一个类型B的声明：
	
```java
Apple a = ...;
Fruit f = a;
```

### 泛型类型的子类型

一个Apple对象的实例可以被赋给一个Fruit对象的声明，但是，一个List<Apple\>却不能赋值给一个List<Fruit\>。

这意味着下面的这段代码是无效的：

```java
List<Apple> apples = ...;
List<Fruit> fruits = apples;
```

**泛型类型跟其是否子类型没有任何关系。**

## 类型擦除

正确理解泛型概念的首要前提是理解类型擦除（type erasure）。

Java中的泛型基本上都是在编译器这个层次来实现的。**在生成的Java字节代码中是不包含泛型中的类型信息的。使用泛型的时候加上的类型参数，会被编译器在编译的时候去掉。这个过程就称为类型擦除**。如在代码中定义的List<Object>和List<String>等类型，在编译之后都会变成List。JVM看到的只是List，而由泛型附加的类型信息对JVM来说是不可见的。

Java编译器会在编译时尽可能的发现可能出错的地方，但是仍然无法避免在运行时刻出现类型转换异常的情况。

类型擦除也是Java的泛型实现方式与C++模板机制实现方式之间的重要区别。

很多泛型的奇怪特性都与这个类型擦除的存在有关，包括：


* 泛型类并没有自己独有的Class类对象。比如并不存在List<String\>.class或是List<Integer\>.class，而只有List.class。

* 静态变量是被泛型类的所有实例所共享的。对于声明为MyClass<T\>的类，访问其中的静态变量的方法仍然是 MyClass.myStaticVar。不管是通过new MyClass<String\>还是new MyClass<Intege\r>创建的对象，都是共享一个静态变量。

* 泛型的类型参数不能用在Java异常处理的catch语句中。因为异常处理是由JVM在运行时刻来进行的。由于类型信息被擦除，JVM是无法区分两个异常类型MyException<String\>和MyException<Integer\>的。对于JVM来说，它们都是 MyException类型的。也就无法执行与异常对应的catch语句。

## 类型通配符

类型通配符一般是使用 ? 代替具体的类型实参。

因为**泛型类型跟其是否子类型没有任何关系。**所以，类似于Box<Number>在逻辑上不能视为Box<Integer>的父类。

```java
public class GenericTest {

	public static void main(String[] args) {

		Box<Number> name = new Box<Number>(99);
		Box<Integer> age = new Box<Integer>(712);

		getData(name);
		
		//The method getData(Box<Number>) in the type GenericTest is 
		//not applicable for the arguments (Box<Integer>)
		getData(age);   // 1

	}
	
	public static void getData(Box<Number> data){
		System.out.println("data :" + data.getData());
	}

}

class Box<T> {

	private T data;

	public Box() {

	}

	public Box(T data) {
		this.data = data;
	}

	public T getData() {
		return data;
	}

}
```

下面的代码是无法编码通过的。

怎么解决上面的问题呢？

这时候可以通过通配符来解决：

```java
public static void getData(Box<?> data) {
	System.out.println("data :" + data.getData());
}
```

##类型通配符上限与下限

类型通配符上限通过形如Box<? extends Number>形式定义，相对应的，类型通配符下限为Box<? super Number>形式，其含义与类型通配符上限正好相反。

上限demo：

```java
public static void getData(Box<? extends Number> data) {
	System.out.println("data :" + data.getData());
}
```

Box<? extends Number>：参数类型Box的泛型必须是Number的子类

下限demo

```java
public static void getData(Box<? super Number> data) {
	System.out.println("data :" + data.getData());
}
```

Box<? super Number>：参数类型Box的泛型必须是Number的父类

## 泛型方法

泛型方法定义如下：

```java
修饰符 <T> T 方法名(Class<T> c){
	方法体
}
```

定义泛型方法时，必须在返回值前边加一个<T>，来声明这是一个泛型方法，持有一个泛型T，然后才可以用泛型T作为方法的返回值。Class<T>的作用就是指明泛型的具体类型，而Class<T>类型的变量c，可以用来创建泛型类的对象。

编写Java泛型方法,返回值类型和至少一个参数类型应该是泛型，而且类型应该是一致的。

下面主要介绍两种十分相似的java泛型方法的使用以及它们之间的区别。


```java
public static <T extends CommonService> T getService(Class<T> clazz) {
	T service = (T) serviceMap.get(clazz.getName());
	if (service == null) {
		service = (T) ServiceLocator.getService(clazz.getName());
		serviceMap.put(clazz.getName(), service);
	}
	return service;
}

```

```
public static <T> T getService(Class<? extends CommonService> clazz) {
	T service = (T) serviceMap.get(clazz.getName());
	if (service == null) {
		service = (T) ServiceLocator.getService(clazz.getName());
		serviceMap.put(clazz.getName(), service);
	}
	return service;
}
```

这两个泛型方法只有方法的签名不一样，方法体完全相同。

第一种泛型方法：返回值和参数值的类型是一致，推荐使用;

第二种泛型方法：返回值和参数值的类型不是一致，请谨慎或避免使用。
