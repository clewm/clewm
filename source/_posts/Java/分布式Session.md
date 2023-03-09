---
title: 分布式Session

categories: 
- 技术
- 分布式

tags:
- Java
- 分布式

description: 分布式Session的生成与使用

cover: https://md-1259549904.cos.ap-shanghai.myqcloud.com/img/4k%20%E5%88%9D%E9%9F%B3%20%E5%A5%B3%E5%AD%A9%20%E7%AC%9B%E5%AD%90%20%E9%95%BF%E5%8F%91%20%E9%95%BF%E8%A3%99%E5%AD%90%20%E5%8A%A8%E6%BC%AB%E5%A3%81%E7%BA%B8_%E5%BD%BC%E5%B2%B8%E5%9B%BE%E7%BD%91.jpg
---

![4k 初音 女孩 笛子 长发 长裙子 动漫壁纸_彼岸图网](https://md-1259549904.cos.ap-shanghai.myqcloud.com/img/4k%20%E5%88%9D%E9%9F%B3%20%E5%A5%B3%E5%AD%A9%20%E7%AC%9B%E5%AD%90%20%E9%95%BF%E5%8F%91%20%E9%95%BF%E8%A3%99%E5%AD%90%20%E5%8A%A8%E6%BC%AB%E5%A3%81%E7%BA%B8_%E5%BD%BC%E5%B2%B8%E5%9B%BE%E7%BD%91.jpg)

# 分布式Session

<img src="https://md-1259549904.cos.ap-shanghai.myqcloud.com/img/image-20220720101945526.png" alt="image-20220720101945526" style="zoom: 200%;" />

## Session

### 本质：

**session技术就是一种基于后端有别于数据库的临时存储数据的技术**



### 存活时间：

可以通过如下来设置一次Session的存活时间，在这个时间内若再次发送请求，则Session的存活时间将会刷新，若超过时间无请求发送，再次发送的时候会再次创建一次新的Session会话。

```java
HttpSession session = request.getSession();
session.setMaxInactiveInterval(2); //单位是秒

session.getId(); //查看session的唯一标识
```

### 实现

添加redis官方的实现依赖即可，可以在配置类上添加如下注解，配置相关参数

```xml
        <dependency>
            <groupId>org.springframework.session</groupId>
            <artifactId>spring-session-data-redis</artifactId>
            <version>2.7.0</version>
        </dependency>
```

可以在配置类上添加如下注解，配置相关参数

```java
@EnableRedisHttpSession(maxInactiveIntervalInSeconds = 3600 #Session的存活时间,redisNamespace = "session_info存储在redis中的命名空间",flushMode = FlushMode.IMMEDIATE #刷新策略,saveMode = SaveMode.ON_SET_ATTRIBUTE #刷新策略)
```

#### 刷新策略

- ON_SAVE: 只有当SessionRepository.save(Session)方法被调用时,才会将session中的数据同步到redis中. 在web 应用中, 当请求完成响应后, 才开始同步. 也就是说在执行response之前session数据都是缓存在本地的.
- IMMEDIATE: 实时同步session 数据到redis. 当执行SessionRepository.createSession()时, 会将session数据同步到redis中;当对session的attribute进行set/remove 等操作时, 也会同步session中的数据到redis中.

#### 保存策略

保存Session属性更改的时机，是调用Set属性方法时保存还是Get属性方法调用的时候保存，还是总是保存，一般默认即可。