---
title: Spring Bean Lifecycle
author: Jiaqi Gao
description: 
date: 2023-06-02T23:34:23+08:00
lastmod: 2023-06-02T23:34:23+08:00
slug: spring-bean-lifecycle
image: https://cdn.jsdelivr.net/gh/Vcancel/uPicRepo@main/uPic/Bean的生命周期.png
math: false
license: 
hidden: false
comments: true
draft: false
categories:
  - Spring Framework
tags:
  - Spring
series:
aliases:
---

## 简介

Bean的生命周期主要包含**实例化**、**属性注入**、**初始化**、**使用**和**销毁**几个阶段。

## Bean的生命周期

#### 简略描述

1. Spring启动，查找并加载需要被Spring管理的bean，进行Bean的实例化。
2. Bean实例化后对将Bean的引入和值注入到Bean的属性中。
3. 如果Bean实现了`BeanNameAware`接口，Spring将Bean的Id传递给`setBeanName()`方法。
4. 如果Bean实现了`BeanFactoryAware`接口，Spring将调用`setBeanFactory()`方法，将`BeanFactory`容器实例传入。
5. 如果Bean实现了`ApplicationContextAware`接口的话，Spring将调用Bean的`setApplicationContext()`方法，将Bean所在应用上下文引用传入。
6. 如果Bean实现了`BeanPostProcessor`接口，Spring就将调用他们的`postProcessBeforeInitialization()`方法。
7. 如果Bean实现了`InitializingBean`接口，Spring将调用他们的`afterPropertiesSet()`方法。
8. 如果Bean使用`init-method`声明了初始化方法，该方法也会被调用。
9. 如果Bean实现了`BeanPostProcessor`接口，Spring就将调用他们的`postProcessAfterInitialization()`方法。
10. Bean已经准备就绪，可以被应用程序使用了。Bean将一直驻留在应用上下文中，直到应用上下文被销毁。
11. 如果bean实现了`DisposableBean`接口，Spring将调用它的`destroy()`接口方法。
12. 如果bean使用了`destroy-method`声明销毁方法，该方法也会被调用。

#### 完整的生命周期

![Bean的生命周期](https://cdn.jsdelivr.net/gh/Vcancel/uPicRepo@main/uPic/Bean的生命周期.png)

###### 初始化

* `BeanNameAware.setBeanName()`：在创建此Bean的Bean工厂中设置Bean的名称，在普通属性设置之后调用，在`InitializinngBean.afterPropertiesSet()`方法之前调用。
* `BeanClassLoaderAware.setBeanClassLoader()`：在普通属性设置之后，`InitializingBean.afterPropertiesSet()`之前调用。
* `BeanFactoryAware.setBeanFactory()`：回调提供了自己的Bean实例工厂，在普通属性设置之后，在`InitializingBean.afterPropertiesSet()`或者自定义初始化方法之前调用。
* `EnvironmentAware.setEnvironment()`：设置`environment`在组件使用时调用。
* `EmbeddedValueResolverAware.setEmbeddedValueResolver()`：设置`StringValueResolver` 用来解决嵌入式的值域问题。
* `ResourceLoaderAware.setResourceLoader()`：在普通bean对象之后调用，在`afterPropertiesSet`或者自定义的`init-method`之前调用，在`ApplicationContextAware`之前调用。
* `ApplicationEventPublisherAware.setApplicationEventPublisher()`：在普通Bean属性之后调用，在初始化调用`afterPropertiesSet`或者自定义初始化方法之前调用。在`ApplicationContextAware`之前调用。
* `MessageSourceAware.setMessageSource()`：在普通bean属性之后调用，在初始化调用`afterPropertiesSet`或者自定义初始化方法之前调用，在`ApplicationContextAware`之前调用。
* `ApplicationContextAware.setApplicationContext()`：在普通Bean对象生成之后调用，在`InitializingBean.afterPropertiesSet`之前调用或者用户自定义初始化方法之前。在`ResourceLoaderAware.setResourceLoader`，`ApplicationEventPublisherAware.setApplicationEventPublisher`，`MessageSourceAware`之后调用。
* `ServletContextAware.setServletContext()`：运行时设置`ServletContext`，在普通Bean初始化后调用，在`InitializingBean.afterPropertiesSet`之前调用，在 `ApplicationContextAware` 之后调用。
> **注：是在WebApplicationContext 运行时**
* `BeanPostProcessor.postProcessBeforeInitialization()`：将此`BeanPostProcessor`应用于给定的新Bean实例，在任何Bean初始化回调方法（像是`InitializingBean.afterPropertiesSet`或者自定义的初始化方法）之前调用。这个Bean将要准备填充属性的值。返回的Bean示例可能被普通对象包装，默认实现返回是一个Bean。
* `BeanPostProcessor.postProcessAfterInitialization()`： 将此`BeanPostProcessor`应用于给定的新Bean实例，在任何Bean初始化回调方法（像是`InitializingBean.afterPropertiesSet`或者自定义的初始化方法）之后调用。这个Bean将要准备填充属性的值。返回的Bean示例可能被普通对象包装。
* `InitializingBean.afterPropertiesSet()`：被`BeanFactory`在设置所有Bean属性之后调用（并且满足`BeanFactory`和`ApplicationContextAware`）。

###### 销毁

* `DestructionAwareBeanPostProcessor.postProcessBeforeDestruction()`：在销毁之前将此`BeanPostProcessor`应用于给定的Bean实例。能够调用自定义回调，像是`DisposableBean`的销毁和自定义销毁方法，这个回调仅仅适用于工厂中的单例Bean（包括内部Bean）
* 实现了自定义的`destory()`方法。