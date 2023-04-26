---
title: '从Spring到Spring Boot'
date: 2023-01-07T10:54:37+08:00
draft: false
---

从 Spring 到 Spring Boot，迁移升级使用过程，以及各种踩坑记录。

## 概述

从 Spring 到 Spring Boot，整体开发、运行方式主要变化。

| -           | 当前模式             | 新模式（本地）              | 新模式（线上）                |
| ----------- | -------------------- | --------------------------- | ----------------------------- |
| 开发习惯    | Spring + 外置 Tomcat | Spring Boot（embed tomcat） | Spring Boot War + 外置 Tomcat |
| Java 版本   | 8、11、16、17        | 11、17(推荐)                | 11、17(推荐)                  |
| Tomcat 版本 | 8.x、9.x             | 9.x                         | 9.x                           |

说明：

1. 理论上支持 Java11，但是要求业务方尽量使用 Java17。
2. 线上运行支持 Spring Boot jar 直接运行，但只开放给部分业务组，主要业务仍以 war + tomcat 为主。

## 快速开始

1. 线下支撑系统导航，点击 `脚手架` 进入 spring start 页面，按自己需求选择模块，生成自己业务模式初始化代码。
2. 写（Copy）业务代码到项目里，修改 pom.xml 根据需要添加新的依赖。
3. 查看本文档中 `遇见问题及解决方案` 章节，注意如果是老项目迁移，这一步很重要。
4. 本地开发工具启动 main 方法。
5. 上线发布系统，选择 `tomcat9:openjdk17` 镜像，并勾选 `镜像JDK版本编译代码`。

以上生成的一个最简略的代码结构，更多复杂使用方式参考下方主要 starter 使用说明。

## 主要 starter 使用说明

文档会延后，代码不会骗人，更多说明参考各个项目源码的 README，README 会实时更新。

### fxiaoke-spring-cloud-parent

目前有两个公司级父 pom：

1. 新：com.fxiaoke.cloud.fxiaoke-spring-cloud-parent 用于 Spring Boot/Cloud 方式开发。
2. 旧：com.fxiaoke.common.fxiaoke-parent-pom 用于原纯 Spring + Tomcat 方式开发。

注意：

1. 这两个 pom 仍然都会更新，但不是实时同步，新 pom 更新一般比较晚。
2. 旧 pom 区分线上和线下版本，新 pom 目前只有一份并不区分。

Maven 项目 parent 统一使用公司新 parent pom，这里定义了 Spring Boot、Spring Cloud 以及内部定制的各种 support 和 starter 版本号。

```xml
<parent>
  <groupId>com.fxiaoke.cloud</groupId>
  <artifactId>fxiaoke-spring-cloud-parent</artifactId>
  <!-- 注意使用最新版本 -->
  <version>1.1.0-SNAPSHOT</version>
 <relativePath/>
</parent>
```

主要升级项需关注：

- Spock and Groovy：Spock 由原来的 1.x 升级到 2.x 版本，同时 Groovy 升级到 4.x 版本，Junit4 升级到 Junit5。

已知废弃依赖：

| 废弃项               | 替代项                 | 说明                                    |
| -------------------- | ---------------------- | --------------------------------------- |
| javax.annotation-api | jakarta.annotation-api | 随 spring boot 版本走，且两个包不能共存 |

## spring-boot-starter-actuator

目前强制依赖 `spring-boot-starter-actuator`，容器镜像里使用它实现健康检查。
另外强制依赖 `spring-boot-starter-web`，因为有些基础组件依赖了 `ServletContext`。

注意：
actuator 的引入会带来一些额外收益（或者叫坑），之前我们健康检测只检查服务端口活着，而 actuator 默认还额外检查各个中间件的状态，比如 ES 连接失败健康检查也会失败。如果业务方不希望检查，需要主动 exclude，具体方式和更多高级应用参考 spring-boot-starter-actuator 官方文档。

## cms-spring-cloud-starter

配置中心 starter，类似 spring-cloud-consul/nacos/config，对接配置中心，实现配置文件动态加载、刷新，代替原 ReloadablePropertySourcesPlaceholderConfigurer。

使用步骤：

1. 引入 starter。

   ```xml
        <dependency>
        <groupId>com.fxiaoke.cloud</groupId>
        <artifactId>cms-spring-cloud-starter</artifactId>
        <!-- 版本号建议不写，使用parent定义好的版本 -->
        </dependency>
   ```

2. 增加 src/main/resources/application.properties 文件，内容如下。

   ```properties
   # 当前模块名，必填，必须全局唯一，一般和maven子模块保持一致
   spring.application.name=cms-starter-sample
   # 配置导入，这一行必须写。但是配置文件本身是否必须是通过optional控制的
   spring.config.import=optional:cms:${spring.application.name}
   ```

   我们使用 `spring.config.import` 固定格式为 `optional:cms:file-name`。
   optional 表示这个文件可选，配置中心不存在的时候也允许启动，`cms` 是固定字符代表对接 fs 配置中心。
   同时支持多个，多个如果想写在一行，用分号分割。例如：

   ```properties
   spring.config.import=optional:cms:${spring.application.name};cms:dubbo-common
   ```

3. 在 CMS 配置中心创建需要的配置文件，文件名为 `spring-cloud-${spring.application.name}`，其中${spring.application.name}替换成真正的文件名，注意当前版本自动追加了前缀`spring-cloud-`且不允许修改。
4. 代码中使用几种方式参考 sample 代码，文档查看 spring 官方`ConfigurationProperties`和 `@Value` 说明。
5. 配置变更后，如果想响应变更事件，实现自己逻辑，自定义类中`implements ApplicationListener<RefreshScopeRefreshedEvent>`
6. 配置加解密，在配置中心中有个加密功能框（如果看不到可能是没有权限），先使用本 starter 的秘钥加密（注意：本 starter 使用了单独的秘钥），使用固定格式 `ENC(加密后的内容)`配置到文件里，在 java 里 get value 就是已经解密后的了。例如：

   ```properties
   sample.sensitive=ENC(30E239E0958AF3179C7E8EBA3DF618FD)
   ```

响应配置更新：

1. 对于使用使用 ConfigurationProperties 映射的对象类，从对象中每次 get 的值都是刷新后的。推荐这种方式。

   ```java
    @Data
    @Configuration
    @ConfigurationProperties(prefix = "sample")
    public class SampleProperties {
        private String name;
    }
   ```

2. `@RefreshScope +  @Value` 获取 Value 注解的新值。

   ```java
   @Service
   @RefreshScope
   public class ValueService {
    @Value("${sample.over.value}")
    @Getter
    private String watchValue;
   }
   ```

3. 监听 RefreshScopeRefreshedEvent 事件。

   ```java
   @EventListener(RefreshScopeRefreshedEvent.class)
   public void handlerPropertiesChangeEvent(RefreshScopeRefreshedEvent event) {
    //此时配置Bean已刷新完成，处理自己的业务逻辑
   }

   ```

## 老项目迁移升级

1. POM 迁移，依赖项可能被开源软件主动提升版本，要注意版本变化。已知的组件会通过 fxiaoke parent 控制。
2. 原有的 xml 配置，可以改为注解形式，也可以不改直接 `@ImportResource` 使用。
3. 注意配置扫描范围，原来 xml 中可能是配置是某几个包，Spring Boot 默认扫描 Application.java 所在包，范围可能扩大。
4. 原来 tomcat web.xml 相关配置，尤其是各种 filter、servlet，需要迁移。包括我们自定义的一些工具。
5. Unit Test 更换注解，目前默认 junit 版本是 junit5，原 junit4 注解位置变更。

### 迁移辅助

辅助工具： 使用 [EMT4J](https://github.com/adoptium/emt4j) 提前扫描，通过静态检测指导从 Java 8 升级到 Java 17 需要注意的变更项。

### War 配置转移

If you try to migrate a Java legacy application to Spring Boot you will find out that [Spring Boot ignores
the web.xml file](https://github.com/spring-projects/spring-boot/issues/2175) when it is run as embedded container.

webapp web.xml 配置如何转移到 spring boot war 形式。
参考：<https://www.baeldung.com/spring-boot-dispatcherservlet-web-xml>

## 已知限制

目前发现几个体验不太好的地方，正在想办法优化。

- 引入我们的 support，常常触发 Spring Auto Config（尤其是 ConditionOnClass 类型的），而我们的 support 有些未适配成 starter，需要开发人员主动排除默认的实现。

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

- 我服了！SpringBoot 升级后这服务我一个星期都没跑起来：
  <https://www.toutiao.com/article/7163602391366074916/?app=news_article&timestamp=1667992250&use_new_style=1&req_id=20221109191050010158031044021CE00D&group_id=7163602391366074916&share_token=A8B008C4-29B7-4ADD-8636-3A17BA91A3BA&tt_from=weixin&utm_source=weixin&utm_medium=toutiao_ios&utm_campaign=client_share&wxshare_count=1&source=m_redirect&wid=1668061929517>