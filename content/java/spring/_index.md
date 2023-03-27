---
title: '从Spring到Spring Cloud'
date: 2023-01-07T10:54:37+08:00
draft: false
---

从 Spring 到 Spring Cloud，迁移升级使用指导，以及各种踩坑记录。

## 整体目标

日常开发直接 run Spring Boot main 方法启动 embed tomcat，和开源方式一样。

运行方式：

1. 目前是打包成 war 以 Tomcat 9.x 版本运行。
2. 暂只支持 Java 17 一个版本，如果业务有强烈需要再支持其他版本。

## 业务方使用

发布系统镜像：选择 tomcat9 系列。

## Spring Cloud 组件

配置管理：
使用目前 CMS 配置中心，支持原有 ReloadablePropertyPostProcessor，另外定制 cms-starter，支持开源 property source 方式管理配置（推荐大家逐渐往这种方式迁移），支持配置自动刷新。
定制的这个 cms-starter 只支持 properties 格式。
配置中心支持自动加解密，定制的 starter 里实现（可参考开源项目<https://github.com/ulisesbocchio/jasypt-spring-boot>）。

application.propeties 规划：在配置中心，application.propeties 通过 include 的方式引入 “通用+业务自定义”，通用包含 ES、Redis、Kafka 等等各通用组件定义如果有改动只需要改通用定义，各业务单独自定义一个文件。对客户端而言，拉取的就是一个 application 文件。各属性增加前缀，保证 key 不重复。

服务注册发现：
Nacos 作为 Server，新项目直接使用 Spring Cloud Alibaba Nacos，老项目使用 nacos-support。

Http Client：
原 http-spring-support 继续保留沿用，老项目和迁移过渡期间使用。
新服务希望能基于 Spring Cloud OpenFeign/RestTemplate + Loadbalancer，做一些定制融合我们的服务发现、路由、埋点、流量摘除。

熔断限流降级：
使用 Spring Cloud Alibaba Sentinel。
目前我们有些 support 里已经内置了 Resilience4J，短期内替换不掉，仍然可用。

分布式事务：
使用 Spring Cloud Alibaba Seata。业务团队根据需要自行引入。

各种数据库中间件：
目前的 redis、es、kafka supoort 仍然需要支持，否则业务侧代码改动较大。封装成 starter 一键引入和配置。
开源方式使用起来并不冲突，但是缺少 Trace 等一些定制化内容，逐渐封装增加切入点向开源方式靠近。

分布式任务调度：
继续使用 xxl-job。

网关：
暂不动。后续考虑 Spring Cloud Gateway or Service Mesh。

测试框架：
spock + Junit5，根据业务现有运行模式自行选择，默认不引入 junit4，如果有需要的自行引入。

其他：
spring-boot-actuator: 可代替健康检查，补充 logging 管理代替目前 logconf，补充部分 metrics。
Logging：外置 fluent-bit 负责全部的日志采集，替换掉现有 logback appender，制定日志规范。
Metrics：现有 CountService 能否切换到 Open Metrics，用开源替代。

## Starter 迁移

父 POM 规划：
新建 fs spring-cloud-parent，原 partent pom 保留给老项目用，这两个 pom 共存挺长一段时间。
新 pom 会 import spring-cloud-dependencies 和 spring-cloud-alibaba-dependencies。
新 pom 里涵盖原有 pom 的各个组件定义，但是组件版本`能升尽升`（开源组件随 spring-cloud-dependencies）。例外：原来有定制的不升级，比如 dubbo、druid，强依赖环境的组件不升级，比如 kafka、ES 和 server 都有配套关系。

目前已封装的 Starter 迁移指导：
xxx：xxx 示例 es starter。

## 开发规范

- 强制启用 Spring Cloud bootstrap，bootstrap.properties 必须包含如下项：

```properties
# name在很多开源组件中都有严格的语义，需要指定且全局唯一不能变
spring.application.name=your-unique-name
```

- 应用名：包含业务语义且全局唯一。

### 包命名规范

类名唯一避免 Jacoco 扫描覆盖。

### Test 规范

要求 Test 默认必须能跑通过。

### Spring

所有 spring 的包必须由 spring starter 引入，类似以下这种主动直接删掉。
AspectJ 通过 spring-boot-starter-aop 引入。

```xml
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context-support</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>${spring.version}</version>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjrt</artifactId>
            <version>${aspectj.version}</version>
        </dependency>
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>${aspectj.version}</version>
        </dependency>
```

### 日志规范

pom log 引入，去掉所有主动引入的 logback 和 slf4j 相关依赖，由 spring-boot-starter-logging 自动引入，类似以下这种直接删掉（注意：如果某模块有对 logback 的定制，注意验证兼容性）。

```xml
    <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.13</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>log4j-over-slf4j</artifactId>
            <version>1.7.13</version>
        </dependency>
                <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-core</artifactId>
            <version>1.1.7</version>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>1.1.7</version>
        </dependency>
```

Logback 配置文件统一将 logback.xml 改为 logback-spring.xml，内容约束如下：

日志打印规范。

## 老项目迁移升级

1. POM 迁移，依赖项可能被开源软件主动提升版本，要注意版本变化。已知的组件会通过 fxiaoke parent 控制。
2. 原有的 xml 配置，可以改为注解形式，也可以不改直接 import 使用。
3. 注意配置扫描范围，原来 xml 中可能是配置是某几个包，Spring Boot 默认扫描 Application.java 所在包，范围可能扩大。
4. 原来 tomcat web.xml 相关配置，尤其是各种 filter、servlet，需要迁移。包括我们自定义的一些工具。
5. Unit Test 更换注解，目前默认 junit 版本是 junit5，原 junit4 注解位置变更。

### 迁移辅助

参考文章：
辅助工具：EMT4J 使用，静态检测指导从 Java 8 升级到 Java 17，发布系统做入口。

### War 配置转移

If you try to migrate a Java legacy application to Spring Boot you will find out that [Spring Boot ignores
the web.xml file](https://github.com/spring-projects/spring-boot/issues/2175) when it is run as embedded container.

webapp web.xml 配置如何转移到 spring boot war 形式。
参考：<https://www.baeldung.com/spring-boot-dispatcherservlet-web-xml>

### JSP

好吧，我们还有 JSP。
TODO： 404

### CMS 配置文件迁移方式

参考新配置文件使用规范。

原 spring-cms.xml 典型配置如：

```xml
    <bean class="com.github.autoconf.spring.reloadable.ReloadablePropertyPostProcessor" c:placeholderConfigurer-ref="autoConf"/>
    <bean id="autoConf" class="com.github.autoconf.spring.reloadable.ReloadablePropertySourcesPlaceholderConfigurer"
          p:fileEncoding="UTF-8"
          p:ignoreResourceNotFound="true"
          p:ignoreUnresolvablePlaceholders="false"
          p:location="classpath:application.properties"
          p:configName="online-manage-provider"/>
```

使用举例和迁移指导，请参考资料中迁移指导。

## 已知限制

目前发现几个体验不太好的地方，正在想办法优化。

- 引入我们的 support，常常触发 Spring Auto Config（尤其是 ConditionOnClass 类型的），而我们的 support 有些未适配成 starter，需要开发人员排除默认的实现。

## 遇见问题及解决方案

- PostConstruct 和 PreDestroy 注解不生效

原因：PostConstruct、PreDestroy 等注解可能存在多个实现或者过个版本，比如以下 jar 包都可能包含：

```console
        javax.annotation-api-1.3.2.jar
        jakarta.annotation-api-1.3.5.jar
        jboss-annotations-api_1.3_spec-2.0.1.Final.jar
```

解决方法：排除依赖，只保留 jakarta.annotation-api 一种，且只能有一个版本。

- kafka 使用报错，日志类似如下：

```log
ERROR c.f.s.SenderManager cannot send, org.apache.kafka.common.KafkaException: org.apache.kafka.clients.producer.internals.DefaultPartitioner is not an instance of org.apache.kafka.clients.producer.Partitioner
```

原因：因为 classpath 下包含多个不同版本的 kafka-client.jar，检查依赖项，确保只引用一个版本。

- 告警：SLF4J: Class path contains multiple SLF4J bindings.

多个 jar 包含 SLF4J 实现，或引入了多个 logback 版本，请根据提示排除不需要的 jar 包。

- XML 中使用 AOP 注解，运行期报错如下（建议用到 AOP 的提前检查，因为运行期才会报错）：JoinPointMatch ClassNotFoundException

```log
 Caused by: java.lang.ClassNotFoundException: org.aspectj.weaver.tools.JoinPointMatch
  at org.apache.catalina.loader.WebappClassLoaderBase.loadClass(WebappClassLoaderBase.java:1412)
  at org.apache.catalina.loader.WebappClassLoaderBase.loadClass(WebappClassLoaderBase.java:1220)
  ... 58 more
```

依赖 spring aop，请确认是否引入 `spring-boot-starter-aop`。

- 本地使用 Java 17 启动，类似如下报错。

```log
ERROR o.s.b.SpringApplication Application run failed java.lang.reflect.InaccessibleObjectException: Unable to make protected final java.lang.Class java.lang.ClassLoader.defineClass(java.lang.String,byte[],int,int,java.security.ProtectionDomain) throws java.lang.ClassFormatError accessible: module java.base does not "opens java.lang" to unnamed module @443118b0
        at java.base/java.lang.reflect.AccessibleObject.checkCanSetAccessible(AccessibleObject.java:354)
        at java.base/java.lang.reflect.AccessibleObject.checkCanSetAccessible(AccessibleObject.java:297)
        at java.base/java.lang.reflect.Method.checkCanSetAccessible(Method.java:199)
        at java.base/java.lang.reflect.Method.setAccessible(Method.java:193)
        at com.alibaba.dubbo.common.compiler.support.JavassistCompiler.doCompile(JavassistCompiler.java:123) [6 skipped]
        at com.alibaba.dubbo.common.compiler.support.AbstractCompiler.compile(AbstractCompiler.java:59)
        at com.alibaba.dubbo.common.compiler.support.AdaptiveCompiler.compile(AdaptiveCompiler.java:46)
```

本地命令行中启动参数里主动追加以下参数（这些参数在发布系统的镜像里默认已经加了），IDEA 启动是设置到`VM options`里：

```bash
--add-opens=java.base/java.lang.reflect=ALL-UNNAMED --add-opens=java.base/java.math=ALL-UNNAMED --add-opens=java.base/java.lang=ALL-UNNAMED --add-opens=java.base/java.io=ALL-UNNAMED --add-opens=java.base/java.util=ALL-UNNAMED --add-opens=java.base/java.util.concurrent=ALL-UNNAMED --add-opens=java.rmi/sun.rmi.transport=ALL-UNNAMED
```

- Bean 重复定义错误，报错信息类似如下。

```log
The bean 'eieaConverterImpl', defined in class path resource [spring/ei-ea-converter.xml], could not be registered. A bean with that name has already been defined in class path resource [spring/ei-ea-converter.xml] and overriding is disabled.
Action:
Consider renaming one of the beans or enabling overriding by setting spring.main.allow-bean-definition-overriding=true
```

可能因为注解扫描范围增广或者有同样包多版本引入，导致扫描到多个。确认多处定义是否一致，如果不一致查看原项目哪个生效，以生效为准。如果一致，找到定义的地方查看是否能整个文件排除掉，实在不能再设置 spring.main.allow-bean-definition-overriding=true。

- 如下报错 `class xxx is not visible from class loader`，常见于 dubbo 服务。
  解决办法：不要用 spring-boot-devtools。 参考链接：<https://blog.csdn.net/zhailuxu/article/details/79305661>
- dubbo 服务 `java.io.IOException: invalid constant type: 18`，日志类似如下：

```console
Wrapped by: java.lang.IllegalStateException: Can not create adaptive extenstion interface com.alibaba.dubbo.rpc.Protocol, cause: java.io.IOExc
eption: invalid constant type: 18
    at com.alibaba.dubbo.common.extension.ExtensionLoader.createAdaptiveExtension(ExtensionLoader.java:723)
    at com.alibaba.dubbo.common.extension.ExtensionLoader.getAdaptiveExtension(ExtensionLoader.java:455)
    ... 29 common frames omitted
Wrapped by: java.lang.IllegalStateException: fail to create adaptive instance: java.lang.IllegalStateException: Can not create adaptive extens
tion interface com.alibaba.dubbo.rpc.Protocol, cause: java.io.IOException: invalid constant type: 18
    at com.alibaba.dubbo.common.extension.ExtensionLoader.getAdaptiveExtension(ExtensionLoader.java:459)
    at com.alibaba.dubbo.config.ServiceConfig.<clinit>(ServiceConfig.java:51)
    ... 28 common frames omitted
```

原因：缺少 javassist 或 javassist 版本太低。目前可用的版本是 `javassist:javassist:3.27.0-GA`。

- Spring Auto Configuration 常见排除：

```java
@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class, MongoDataAutoConfiguration.class})
```

## 参考资料

Spring 升级教训：
<https://www.toutiao.com/article/7163602391366074916/?app=news_article&timestamp=1667992250&use_new_style=1&req_id=20221109191050010158031044021CE00D&group_id=7163602391366074916&share_token=A8B008C4-29B7-4ADD-8636-3A17BA91A3BA&tt_from=weixin&utm_source=weixin&utm_medium=toutiao_ios&utm_campaign=client_share&wxshare_count=1&source=m_redirect&wid=1668061929517>
