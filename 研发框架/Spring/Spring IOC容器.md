## 1、什么是 IoC 容器？

IoC（Inverse of Control） 是 Spring 中一个非常重要的概念，是 Spring 框架核心中的核心。IoC，控制反转。简单来说，就是将对象的创建权力由程序反转给 Spring 框架。

### 1.1、容器介绍

`org.springframework.context.ApplicationContext`接口代表了 IoC 容器，它负责实例化，配置和装配 beans。容器通过读取配置元数据获取对象的实例化、配置和装配的描述信息。配置元数据可以通过 `XML`、`Java 注解`、`Java 代码` 来表示。**IoC容器本质就是一个 Map 存储结构。**

`org.springframework.beans`和`org.springframework.context`是Spring IoC 容器的基础，[`BeanFactory`](http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/beans/factory/BeanFactory.html)接口提供一种高级的配置机制能够管理任何类型的对象。[`ApplicationContext`](http://ifeve.com/spring-ioc-1-2/href="http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/context/ApplicationContext.html)是`BeanFactory`的子接口，它添加了如下功能：

- 集成 Spring AOP 更简单
- 消息资源处理（比如在国际化中使用）
- 事件发布
- 特定的应用层 contexts，像 web 应用中的 `WebApplicationContext`

总之，`BeanFactory`提供了配置框架和基本方法，`ApplicationContext`添加更多的企业特定的功能。

### 1.2、容器概述

Spring 提供了几个 `ApplicationContext` 接口的实现类。在独立应用程序中，我们通常使用`ClassPathXmlApplicationContext` 或`FileSystemXmlApplicationContext`来创建实例对象。

下图是 Spring 如何工作的高级展示。你应用中所有的类都由元数据组装到一起，所以当  `ApplicationContext` 创建和实例化后，你就有了一个完全配置好的可执行的系统或应用。

![*The Spring IoC container*](https://docs.spring.io/spring/docs/5.1.12.BUILD-SNAPSHOT/spring-framework-reference/images/container-magic.png)

#### 1.2.1、配置元数据

#### 1.2.2、容器实例化

## 

 

