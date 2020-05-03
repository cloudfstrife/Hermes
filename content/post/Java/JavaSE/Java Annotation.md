---
title: "Java Annotation"
date: 2019-03-04T14:58:17+08:00
categories:
- Java
tags:
- Java
- Annotation
keywords:
- Java
- Annotation
---

从JDK5.0开始，Java增加了Annotation(注解)，Annotation是代码里的特殊标记，这些标记可以在编译、类加载、运行时被读取，并执行相应的处理。通过使用Annotation，开发人员可以在不改变原有逻辑的情况下，在源文件中嵌入一些补充的信息。代码分析工具、开发工具和部署工具可以通过这些补充信息进行验证、处理或者进行部署。 

<!--more-->

## 自定义注解

在定义一个注解之前，先要了解Java为提供的**元注解**和相关定义注解的语法。

### 元注解

元注解的作用就是负责注解其他注解。Java5.0定义了4个标准的meta-annotation类型，它们对其它annotation类型进行注解。

* @Target
* @Retention
* @Documented
* @Inherited

#### @Target

`@Target`用于说明Annotation所修饰的对象范围。（即：被描述的注解可以用在什么地方）
> Annotation可被用于`packages`、`types`（类、接口、枚举、Annotation类型）、类型成员（方法、构造方法、成员变量、枚举值）、方法参数和本地变量（如循环变量、catch参数）。
>
> 在Annotation类型的声明中使用了target可更加明晰其修饰的目标。

`@Target`取值(ElementType)有：

1. **CONSTRUCTOR**：用于描述构造器
2. **FIELD**：用于描述字段声明（包括枚举常量）
3. **LOCAL_VARIABLE**：用于描述局部变量
4. **METHOD**：用于描述方法
5. **PACKAGE**：用于描述包
6. **PARAMETER**：用于描述参数
7. **TYPE**：用于描述类、接口(包括注解类型) 或enum声明

**Demo:**

```java
@Target(ElementType.TYPE)
public @interface DemoAnnotation {
	public String tableName() default "className";		
}
```

```java
@Target(ElementType.FIELD)
public @interface NoDBColumn {

}
```

上面的代码中，注解`Table`可以用于注解类、接口(包括注解类型) 或`enum`声明,而注解`NoDBColumn`仅可用于注解类的成员变量。

#### @Retention

`@Retention`定义了该Annotation的生命周期：某些Annotation仅出现在源代码中，而被编译器丢弃；而另一些却被编译在class文件中；编译在class文件中的Annotation可能会被虚拟机忽略，而另一些在class被装载时将被读取（请注意并不影响class的执行，因为Annotation与class在使用上是被分离的）。

`@Retention`取值（RetentionPoicy）有：

1. `SOURCE`:在源文件中有效（即源文件保留）
2. `CLASS`:在class文件中有效（即class保留）
3. `RUNTIME`:在运行时有效（即运行时保留）

**Demo**

```java
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Column {
	public String name() default "fieldName";
	public String setFuncName() default "setField";
	public String getFuncName() default "getField"; 
	public boolean defaultDBValue() default false;
}
```

`Column`注解的的`RetentionPolicy`的属性值是RUTIME,这样注解处理器可以通过反射，获取到该注解的属性值，从而去做一些运行时的逻辑处理

#### @Documented:

`@Documented`注解表明这个注解应该被`javadoc`工具记录. 默认情况下,`javadoc`是不包括注解的. 但如果声明注解时指定了`@Documented`,则它会被`javadoc`之类的工具处理, 所以注解类型信息也会被包括在生成的文档中。

**Demo**

```java
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Column {
	public String name() default "fieldName";
	public String setFuncName() default "setField";
	public String getFuncName() default "getField"; 
	public boolean defaultDBValue() default false;
}
```

#### @Inherited

`@Inherited`表明该标注是可被继承的。如果一个使用了`@Inherited`修饰的annotation类型被作用于一个class，则这个annotation将可以作用于该class的子类。

> `@Inherited`的Annotation是被标注过的class的子类所继承。类并不从它所实现的接口继承Annotation，方法并不从它所重载的方法继承annotation。
> 
> 当`@Inherited`annotation类型标注的annotation的`Retention`是`RetentionPolicy.RUNTIME`时，则反射API增强了这种继承性。
> 
> 如果使用java.lang.reflect去查询一个`@Inherited`Annotation类型的annotation时，反射代码检查将展开工作：检查class和其父类，直到发现指定的annotation类型被发现，或者到达类继承结构的顶层。

**Demo**

```java
@Inherited
public @interface Greeting {
	public enum FontColor{ BULE,RED,GREEN};
	String name();
	FontColor fontColor() default FontColor.GREEN;
}
```

#### 自定义注解

@interface用来声明一个注解，使用@interface自定义注解时，自动继承了java.lang.annotation.Annotation接口，由编译程序自动完成其他细节。在定义注解时，不能继承其他的注解或接口。其中的每一个方法实际上是声明了一个配置参数。方法的名称就是参数的名称，返回值类型就是参数的类型（返回值类型只能是基本类型、Class、String、enum）。可以通过default来声明参数的默认值。

> 定义注解格式：

```java
public @interface 注解名 {
	定义体
}
```

**注解参数可支持数据类型：**

1. 所有基本数据类型（int,float,boolean,byte,double,char,long,short)
2. String类型
3. Class类型
4. enum类型
5. Annotation类型
6. 以上所有类型的数组

**Annotation类型参数设定**:

1. 成员参数，其修饰符只有public、默认(default)。
1. 参数成员只能用八种基本类型和String、Enum、Class、annotation等数据类型及其数组形式。
1. 当自定义annotation中只有一个参数时，最好将参数名定义为value，因为当参数名为value时，在使用注解的时候可以不指定参数名称而直接赋值即可。
