---
title: SkyWalking

categories: 
- 技术
- 分布式
- 中间件
- 分布式链路追踪

tags:
- Java
- 分布式

description: 分布式链路追踪
---



# SkyWalking

### 下载地址：http://skywalking.apache.org/downloads/

## 概念：

可实现基于Open Tracing规范的分布式链路追踪功能的APM应用性能管理平台

![image-20220823114137847](https://md-1259549904.cos.ap-shanghai.myqcloud.com/img/image-20220823114137847.png)

### UI界面的jar包和配置文件（可修改端  口）

![image-20220823114222953](https://md-1259549904.cos.ap-shanghai.myqcloud.com/img/image-20220823114222953.png)

修改skywalking服务端数据存储方式：

默认是使用H2进行存储，如果服务端重启的话，数据就会丢失，推荐使用es存储，可以在如下位置处进行修改：

![image-20220823134907000](https://md-1259549904.cos.ap-shanghai.myqcloud.com/img/image-20220823134907000.png)

selector处修改为elasticsearch，然后修改es的一些默认配置即可

![image-20220823134723074](https://md-1259549904.cos.ap-shanghai.myqcloud.com/img/image-20220823134723074.png)

### 端口说明

![image-20220823114611913](https://md-1259549904.cos.ap-shanghai.myqcloud.com/img/image-20220823114611913.png)

## 服务接入

比如我需要在window系统上启动一个需要被监控的服务，就需要在这个系统上提前准备一个和skywalking服务端对应版本的skywalking-agent.jar

下载地址：https://archive.apache.org/dist/skywalking/，解压后在根目录即可找到，170多MB的那个

- 如果是通过idea启动的话，在启动配置的VM参数处填入以下：

```shell 
-javaagent:C:\_Code\lcss\lcss-gateway\agent\skywalking-agent.jar #jar包具体位置
-Dskywalking.agent.service_name=lcss-gateway #服务名，最好与nacos中的名字对应
-Dskywalking.collector.backend_service=192.168.1.104:11800 #skywalking服务端暴露的收集数据的端口
```

- 如果是在docker中的话：

同样需要提前准备agent（包含有jar包）

原本项目的启动方式：

```shell
java -jar spring-boot-demo-0.0.1-SNAPSHOT.jar
```

现在变为：

```shell
java -javaagent:/opt/apache-skywalking-apm-bin/agent/skywalking-agent.jar -Dskywalking.agent.service_name=服务名 -Dskywalking.collector.backend_service=127.0.0.1:11800 -jar /opt/spring-boot-demo-0.0.1-SNAPSHOT.jar
```



### 操作如下：

与Src目录平级下建一个agent文件夹，放agent文件夹（从如下路径拷贝来的）

![image-20220916142200142](https://md-1259549904.cos.ap-shanghai.myqcloud.com/img/image-20220916142200142.png)

![image-20220916142128932](https://md-1259549904.cos.ap-shanghai.myqcloud.com/img/image-20220916142128932.png)

文件：

```dockerfile
FROM openjdk:8-jdk-alpine

RUN mkdir -p /lcss-gateway

WORKDIR /lcss-gateway

ARG JAR_FILE=target/lcss-gateway.jar

COPY ${JAR_FILE} app.jar
COPY agent /lcss-gateway/agent ==========>>>>关键,把agent的jar包复制到容器中

EXPOSE 20010

ENTRYPOINT ["java", "-javaagent:/lcss-gateway/agent/skywalking-agent.jar", "-Dskywalking.agent.service_name=lcss-gateway","-Dskywalking.collector.backend_service=192.168.1.104:11800", "-jar","app.jar"]
```

### ps:docker部署启动的时候提示config没找到的踩坑

项目的agent不能只拷贝一个jar包，需要把整个agent文件夹都拷贝进去，因为里面还有配置文件！！

## 日志收集（版本8.4及以上！）

**ps：SpringBoot默认的日志框架是logback，这里介绍的也是基于logback的日志收集方法，可以配合Lombok的@Slf4j正常使用******

依赖：

```xml
<dependency>
    <groupId>org.apache.skywalking</groupId>
    <artifactId>apm-toolkit-trace</artifactId>
    <version>8.5.0</version> =============================该版本最好和skyWalking的版本对应
</dependency>

<dependency>
    <groupId>org.apache.skywalking</groupId>
    <artifactId>apm-toolkit-logback-1.x</artifactId>
    <version>8.5.0</version> =============================该版本最好和skyWalking的版本对应
</dependency>
```



在resource下创建logback-spring.xml文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true" scanPeriod="10 seconds">

    <appender name="stdout" class="ch.qos.logback.core.ConsoleAppender"> ========用来格式化日志输出的
        <encoder class="ch.qos.logback.core.encoder.LayoutWrappingEncoder">
            <layout class="org.apache.skywalking.apm.toolkit.log.logback.v1.x.TraceIdPatternLogbackLayout">
                <Pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%tid] [%thread] %-5level %logger{36} -%msg%n</Pattern>
            </layout>
        </encoder>
    </appender>

    <appender name="grpc" =====================用来上报日志的class="org.apache.skywalking.apm.toolkit.log.logback.v1.x.log.GRPCLogClientAppender">
        <encoder class="ch.qos.logback.core.encoder.LayoutWrappingEncoder">
            <layout class="org.apache.skywalking.apm.toolkit.log.logback.v1.x.mdc.TraceIdMDCPatternLogbackLayout">
                <Pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%X{tid}] [%thread] %-5level %logger{36} -%msg%n</Pattern>
            </layout>
        </encoder>
    </appender>

    <root level="INFO">
        <appender-ref ref="stdout"/> ===================用来格式化日志输出的
        <appender-ref ref="grpc"/> =====================用来上报日志的
    </root>
</configuration>
```

如果服务和skyWalking不在同一个服务器上，还需要在agent文件夹下的config的agent.config中添加如下配置：

```
plugin.toolkit.log.grpc.reporter.server_host=${SW_GRPC_LOG_SERVER_HOST:172.28.231.100}  ===地址
plugin.toolkit.log.grpc.reporter.server_port=${SW_GRPC_LOG_SERVER_PORT:11800} ===端口
plugin.toolkit.log.grpc.reporter.max_message_size=${SW_GRPC_LOG_MAX_MESSAGE_SIZE:10485760} ===最大日志大小
plugin.toolkit.log.grpc.reporter.upstream_timeout=${SW_GRPC_LOG_GRPC_UPSTREAM_TIMEOUT:30} 
```

如何定位某条日志对应的调用链路？

![image-20220918111842645](https://md-1259549904.cos.ap-shanghai.myqcloud.com/img/image-20220918111842645.png)

通过该TID在skyWalking中进行搜索，不支持模糊查询

![image-20220918111916766](https://md-1259549904.cos.ap-shanghai.myqcloud.com/img/image-20220918111916766.png)
