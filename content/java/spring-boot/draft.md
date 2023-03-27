---
title: 'Temp'
date: 2023-01-07T10:54:37+08:00
draft: true
---




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