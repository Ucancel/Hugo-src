---
title: Springboot >> 动态数据源
author: Jiaqi Gao
description: 
date: 2023-06-01T22:40:11+08:00
lastmod: 2023-06-01T22:40:11+08:00
slug: springboot-dynamic-datasource
image: 
math: false
license: 
hidden: false
comments: true
draft: false
categories:
  - Java
  - Spring Framework
tags:
  - Java
  - SpringBoot
series:
aliases:
---

## Dynamic-Datasource

#### 在`pom.xml`中导入依赖
```xml
<dependency>
  <groupId>com.baomidou</groupId>
  <artifactId>dynamic-datasource-spring-boot-starter</artifactId>
</dependency>
```

#### 在`application.yml`中配置动态数据源
```yml
spring:
  datasource:
    dynamic:
      # 设置默认的数据源或者数据源组,默认值即为main
      primary: main
      # 严格匹配数据源,默认false. true未匹配到指定数据源时抛异常,false使用默认数据源
      strict: false
      datasource:
        # 默认数据源
        main: 
          url: url
          username: username
          password: passward
          driver-class-name: xxx.Dirver
        # 其他数据源
        slave_1:
          url: url
          username: username
          password: passward
          driver-class-name: xxx.Dirver
```

#### 在`Service`方法上使用`@DS`注解
```Java
public interface DemoService {
    void testDynamicFunc();
}

@Service
public class DemoServiceImpl implements DemoService {
    @Override
    @DS("main")
    @Transactional(rollbackFor = Exception.class, isolation = Isolation.DEFAULT, propagation = Propagation.REQUIRES_NEW)
    void testDynamicFunc() {
        demoDao.batchInsertOrUpdate(List<Object> record);
    }
}
```

> PS：在使用`@DS`注解时，需要注意以下两点
> * 在同一个`Service`类中，使用多个`@DS`注解配置不同的数据源，可能会失效，解决办法是不同数据源拆分成不同的`Service`类。
> * 在同一个事务中，使用多个`@DS`注解配置不同的数据源，可能会失效，解决办法是将事务的传播方式修改为`propagation = Propagation.REQUIRES_NEW`
