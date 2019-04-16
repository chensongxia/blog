---
title: Spring中的各种Condition注解
date: 2019-04-16 21:54:22
tags:
  - Spring Boot
  - Spring
categories: 
  - Spring
---

## Spring中的各种Condition注解

### @Conditional
@Conditional注解的作用是实现条件配置Bean，当满足条件时配置Bean。当不满足条件时不配置。

```java
@Configuration
@Conditional({Condition1.class,Condition2.class})
public class ConditionTestConfig {

    @Bean
    public String bean1(){
        return new String("bean1");
    }

    @Bean
    public String bean2(){
        return new String("bean2");
    }

}
```
<!-- more -->

@Conditional注解后面的类必须实现Condition接口

```java
public class Condition1 implements Condition {

    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        return false;
    }
}
```

@Conditional可以配置在类上，对这种类中所有@Bean方法生效。也可以配置在方法上，对单一的@Bean方法生效。@Conditional后面可以跟多个实现Condition接口的类，只有当所有类的match方法返回true才注入Bean。如果一个标注@Configuration的类上标注了@Conditional注解，那么对这个类上面的@ComponentScan和@Imoprt对会生效。继承的@Conditional不会生效。

### @ConditionalOnBean
Spring容器中存在相应的Bean才会注入当前Bean。

```java
@Bean
@ConditionalOnBean(name = {"bean1"})
public String bean3() {
    return new String("bean3");
}

@Bean
@ConditionalOnBean(value = {ApplicationContext.class})
public String bean4() {
    return new String("bean3");
}
```

### @ConditionalOnMissingBean
和上面的相反，不存在某个Bean或者某类Bean才注入当前的Bean。

### @ConditionalOnClass
classPath中存在指定的类才会注入当前的Bean。

### @ConditionalOnMissingClass
classpath中不存在指定的类才会注入当前的Bean。

### @ConditionalOnCloudPlatform
只有当指定的Spring Cloud组件激活后才注入当前Bean

### @ConditionalOnEnabledResourceChain
？？

### @ConditionalOnExpression
基于SpEL表达式的条件，条件表达式返回true注入当前Bean。

### @ConditionalOnJava
基于JDK的版本决定是否注入当前的Bean。

### @ConditionalOnWebApplication
如果当前应用是Web应用就注入当前Bean。默认情况下所有Web应用都符合要求，但是我们也可以自己指定类型（Servlet或者Reactive等）。

### @ConditionalOnNotWebApplication
不是Web应用则创建当前Bean。

### @ConditionalOnProperty
根据配置属性值决定是否创建当前Bean。

```java
    @Bean
    @ConditionalOnProperty(prefix = "condition", name = "match",havingValue = "xx",matchIfMissing = true)
    public String bean5(){
        System.out.println("begin to init bean5...");
        return new String("bean5");
    }
```
如果Environment中配置了condition.match这个属性，那么判断这个值是否等于"xx",如果相等则创建Bean，否则不创建。如果Environment中没有配置condition.match属性，则判断matchIfMissing的值，如果等于true则创建当前Bean。

### @ConditionalOnRepositoryType
？？

### @ConditionalOnResource
在classpath中存在指定的资源才创建当前Bean.

```java
@ConditionalOnResource(resources = "classpath:META-INF/services/javax.validation.spi.ValidationProvider")
```

### @ConditionalOnRibbonRestClient
??

### @ConditionalOnSingleCandidate
当Spring容器中只有一个指定的Bean就创建当前Bean