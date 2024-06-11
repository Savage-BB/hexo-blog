---
title: Sping的Bean配置
date: 2023-04-19
description: spring 容器的bean对象可以基于Java代码配置，它像XML配置一样不侵入源代码，同时也支持注解配置
categories: Java
tags: 
  - Spring
---
# Sping的Bean配置

spring 容器的bean对象可以基于Java代码配置，它像XML配置一样不侵入源代码，同时也支持注解配置

# Bean管理

### 元数据

基于Java配置Bean，定义一个类并使用`@configuration`注解作为配置类，在类中的方法上使用`@Bean`注解，就会注册一个以方法返回值为实例的Bean

```java
@Configuration
public class AppConfig {

    @Bean
    public BeanExample beanExample() {
        return new BeanExample();
    }

    @Bean
    public BeanProvider beanProvider() {
        return new BeanProvider();
    }
}
```

默认使用**方法名**作为Bean名称，也可以通过注解的`value`或`name`属性来指定

使用可以使用`AnnotationConfigApplicationContext`来实例化容器

```java
public static void main(String[] args) {
    ApplicationContext applicationContext =
        new AnnotationConfigApplicationContext(AppConfig.class);
    BeanExample beanExample = (BeanExample) applicationContext.getBean("beanExample");
}
```

**Bean的作用域**

在spring配置文件中可以设置bean的作用域

```xml
<bean id="beanExample" class="...xml.BeanExample" scope="prototype"/>
```

使用@Scope注解来指定Bean的作用域

```java
@Bean
@Scope("singleton")
public BeanExample beanExample() {
    return new BeanExample();
}
```

常用的几种Bean作用域如下：

| 作用域 | 描述 |
| --- | --- |
| singleton | （默认）单例作用域，Spring容器内部只创建一个实例 |
| prototype | 原型作用域，容器中创建多个实例，每使用一次会创建一个实例 |
| request | 请求作用域，WEB框架下单次请求内创建一个实例 |
| session | 会话作用域，WEB框架下单次会话内创建一个实例 |
| application | 应用作用域，ServletContext生命周期内创建一个实例 |

### Full模式和Lite模式

一般情况下，@Bean用于@Configuration注解的类下，这种方式为Full模式

当@Bean用于其他注解（）的类下，或者为任何一个Bean内部的方法买这种情况为Lite模式。

```java
// Full
@Configuration
public class AppConfig {

    @Bean
    public BeanExample beanExample() {
        return new BeanExample();
    }
}

// Lite
@Component
public class AppConfig {

    @Bean
    public BeanExample beanExample() {
        return new BeanExample();
    }
}
```

### 组合配置

为了实现模块化配置，可以定义多个配置类，在配置类中使用`@Import`注解来导入其他配置类，在实例化容器的时候，只需要指定AppConfig类，不需要指定所有配置类。

```java
@Configuration
public class OtherConfig {

    @Bean
    public BeanProvider beanProvider() {
        return new BeanProvider();
    }
}

@Configuration
@Import(OtherConfig.class)
public class AppConfig {

    @Bean
    public BeanExample beanExample(BeanProvider beanProvider) {
        BeanExample beanExample = new BeanExample();
        beanExample.setBeanProvider(beanProvider);
        return beanExample;
    }
}
```

### 扫描类路径配置

```java
@Configuration
@ComponentScan(basePackages = "cn.codeartist.spring.bean.java")
public class AppConfig {

}
```

# 依赖管理

### 依赖注入

1. 参数注入：依赖的Bean可以通过方法参数注入
    
    ```java
    @Bean
    public BeanExample beanExample(BeanProvider beanProvider) {
        BeanExample beanExample = new BeanExample();
        beanExample.setBeanProvider(beanProvider);
        return beanExample;
    }
    ```
    
2. 方法注入：同一个配置类中，可以直接调用方法来依赖其他Bean，该方法只在生成CBLIB代理类的Full模式下才生效
    
    ```java
    @Configuration
    public class AppConfig {
    
        @Bean
        public BeanExample beanExample() {
            BeanExample beanExample = new BeanExample();
            beanExample.setBeanProvider(beanProvider());
            return beanExample;
        }
    
        @Bean
        public BeanProvider beanProvider() {
            return new BeanProvider();
        }
    }
    ```
    

### 依赖关系

使用`@DependsOn`注解指定依赖关系

```java
@Bean
@DependsOn("beanProvider")
public BeanExample beanExample() {
    return new BeanExample();
}
```

### 懒加载

使用`@Lazy`注解来配置懒加载

```java
@Lazy
@Bean
public BeanProvider beanProvider() {
    return new BeanProvider();
}
```

懒加载Bean在注入的地方也要加上`@Lazy`注解，或者使用`ApplicationContext.getBean()`方法获取Bean，才能使懒加载生效

```java
@Bean
public BeanExample beanExample(@Lazy BeanProvider beanProvider) {
    BeanExample beanExample = new BeanExample();
    beanExample.setBeanProvider(beanProvider);
    return beanExample;
}
```