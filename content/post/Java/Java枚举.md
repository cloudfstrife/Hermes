---
title: "Java枚举"
date: 2019-03-04T15:04:54+08:00
categories:
- Java
tags:
- Java
- Enum
keywords:
- Java
- Enum
---

枚举（enum）的全称为 enumeration， 是 JDK 1.5  中引入的新特性

<!--more-->

## 语法

创建枚举类型要使用 **enum** 关键字，所创建的类型都是 java.lang.Enum 类的子类。

DEMO:

```
public enum Demo {
	MON, TUE, WED, THU, FRI, SAT, SUN;
}
```

这段代码实际上调用了7次 Enum 的构造函数七次

```	
new Enum<EnumTest>("MON",0);
new Enum<EnumTest>("TUE",1);
new Enum<EnumTest>("WED",2);
new Enum<EnumTest>("THU",3);
new Enum<EnumTest>("FRI",4);
new Enum<EnumTest>("SAT",5);
new Enum<EnumTest>("SUN",6);
```

## 枚举的常用方法

* compareTo(E o)

```
public final int compareTo(E o)
```

> 比较此枚举与指定对象的顺序。在该对象小于、等于或大于指定对象时，分别返回负整数、零或正整数。 枚举常量只能与相同枚举类型的其他枚举常量进行比较。该方法实现的自然顺序就是声明常量的顺序。  

* Class<E> getDeclaringClass()

```
public final Class<E> getDeclaringClass()
```

> 返回与此枚举常量的枚举类型相对应的 Class 对象。当且仅当 e1.getDeclaringClass() == e2.getDeclaringClass() 时，两个枚举常量 e1 和 e2 的枚举类型才相同。（由该方法返回的值不同于由 Object.getClass() 方法返回的值，Object.getClass() 方法用于带有特定常量的类主体的枚举常量。） 

* name()

```
public final String name()
```

> 返回此枚举常量的名称，在其枚举声明中对其进行声明。 与此方法相比，大多数程序员应该优先考虑使用 toString() 方法，因为 toString 方法返回更加用户友好的名称。该方法主要设计用于特殊情形，其正确性取决于获取正确的名称，其名称不会随版本的改变而改变。 

* ordinal()

```
public final int ordinal()
```

> 返回枚举常量的序数（它在枚举声明中的位置，其中初始常量序数为零）。 大多数程序员不会使用此方法。它被设计用于复杂的基于枚举的数据结构，比如 EnumSet 和 EnumMap。 

* toString()

```
public String toString()
```

> 返回枚举常量的名称，它包含在声明中。可以重写此方法，虽然一般来说没有必要。当存在更加“程序员友好的”字符串形式时，应该使用枚举类型重写此方法。 


* static valueOf()

```
static <T extends Enum<T>> T valueOf(Class<T> enumType, String name)
```

> 返回带指定名称的指定枚举类型的枚举常量。

* static values()
	
```
public static <T extends Enum<T>> [] values()
```

> 返回一个包含枚举常量列表的数组

## enum 自定义属性和方法

给 enum 对象加一下 value 的属性和 getValue() 的方法：

```
public enum Demo {

	 MON, TUE, WED, THU, FRI, SAT, SUN;
	
	 private int value;
	 
	 public int getValue(){
		 return value;
	 }		 
}
```

> 枚举像普通的类一样可以添加属性和方法
> 
> 可以为它添加静态和非静态的属性或方法，**枚举写在最前面，否则编译出错**

## 实现带有构造器的枚举

```
public enum Demo {

	MON(0), TUE(1), WED(2), THU(3), FRI(4), SAT(5), SUN(6);

	private int value;
	
	private Demo(int value) {
		this.value = value;
	}
	
	public int getValue() {
		return value;
	}
}
```

> 1. 通过括号赋值,而且必须带有一个参构造器和一个属性跟方法，否则编译出错
> 1. 赋值必须都赋值或都不赋值，不能一部分赋值一部分不赋值；
> 1. 如果不赋值则不能写构造器，否则编译也出错
> 1. 构造器默认也只能是private, 从而保证构造函数只能在内部使用


## EnumSet，EnumMap 的应用

EnumSet Demo:

```
public static void main(String[] args) {
	EnumSet<Demo> demoSet = EnumSet.allOf(Demo.class);
	for (Demo demo : demoSet) {
		System.out.println(demo.toString());
	}
}
```

EnumMap Demo：

```
public static void main(String[] args) {
	
	EnumMap<Demo, String> enumMap = new EnumMap<Demo, String>(Demo.class);
	enumMap.put(Demo.MON, "星期一");
	enumMap.put(Demo.TUE, "星期二");
	enumMap.put(Demo.WED, "星期三");
	enumMap.put(Demo.THU, "星期四");
	enumMap.put(Demo.FRI, "星期五");
	enumMap.put(Demo.SAT, "星期六");
	enumMap.put(Demo.SUN, "星期日");
	
	Iterator<Entry<Demo, String>> it = enumMap.entrySet().iterator(); 
	while (it.hasNext()) {
		Entry<Demo, String> entry = it.next();
		System.out.println(entry.getKey().name() + ":" + entry.getValue());
		
	}
}
```

---

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。