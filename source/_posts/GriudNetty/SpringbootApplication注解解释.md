---
title: SpringbootApplication注解解释
abbrlink: 1471733270
date: 2021-12-12 15:35:01
tags: [Guide,Springboot,Netty,源码阅读]
categories: [Guide,Springboot,Netty,源码阅读]
---
## SpringbootApplication注解解释（Guide）

`@Target：`

> 　　　　　　@Target说明了Annotation所修饰的对象范围：Annotation可被用于 packages、types（类、接口、枚举、Annotation类型）、类型成员（方法、构造方法、成员变量、枚举值）、方法参数和本地变量（如循环变量、catch参数）。在Annotation类型的声明中使用了target可更加明晰其修饰的目标。
>
> 　　　　　作用：用于描述注解的使用范围（即：被描述的注解可以用在什么地方）
>
> 　　　　　取值(ElementType)有：
>
> 　　　　　　　1.CONSTRUCTOR:用于描述构造器
> 　　　　　　　2.FIELD:用于描述域
> 　　　　　　　3.LOCAL_VARIABLE:用于描述局部变量
> 　　　　　　　4.METHOD:用于描述方法
> 　　　　　　　5.PACKAGE:用于描述包
> 　　　　　　　6.PARAMETER:用于描述参数
> 　　　　　　　7.TYPE:用于描述类、接口(包括注解类型) 或enum声明

`@Retention(RetentionPolicy.RUNTIME)`

> 前面有提到注解按生命周期来划分可分为3类：
>
> 1、RetentionPolicy.SOURCE：注解只保留在源文件，当Java文件编译成class文件的时候，注解被遗弃；
> 2、RetentionPolicy.CLASS：注解被保留到class文件，但jvm加载class文件时候被遗弃，这是默认的生命周期；
> 3、RetentionPolicy.RUNTIME：注解不仅被保存到class文件中，jvm加载class文件之后，仍然存在；

`@Documented`

> **@**Documented 注解表明这个注解应该被 javadoc工具记录.  默认情况下,javadoc是不包括注解的. 但如果声明注解时指定了 @Documented,则它会被 javadoc 之类的工具处理,  所以注解类型信息也会被包括在生成的文档中，是一个标记注解，没有成员。

`@Inherited`

> **类继承关系中@Inherited的作用**
>
> 类继承关系中，子类会继承父类使用的注解中被@Inherited修饰的注解
>
> **接口继承关系中@Inherited的作用**
>
> 接口继承关系中，子接口不会继承父接口中的任何注解，不管父接口中使用的注解有没有被@Inherited修饰
>
> **类实现接口关系中@Inherited的作用**
>
> 类实现接口时不会继承任何接口中定义的注解

`@ComponentScan`

这个注解是  Guide 自主实现的，故我们去查询Guide写的类