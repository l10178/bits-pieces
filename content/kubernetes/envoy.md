---
title: 'Envoy生产配置最佳实践'
date: 2023-02-15T21:54:37+08:00
draft: false
---

## Envoy 监控指标

Envoy 将其指标分为以下主要类别：

- Downstream：与进入代理的连接和请求相关的指标。它们由 listener、HTTP connection manager(HCM)、TCP proxy filter 等产生。
- Upstream：与传出连接和代理发出的请求相关的指标。它们由 connection pool、router filter、tcp proxy filter 等产生。
- Server：描述 Envoy 内部状态的指标。其中包括正常运行时间或分配的内存等指标。

Envoy 发出三种指标数据类型：

- Counter(Counters)：无符号整数，只会增加而不会减少。例如，总请求。
- 仪表(Gauges)：增加和减少的无符号整数。例如，当前活动的请求。
- Histograms(Histograms)：作为指标流的一部分的无符号整数，然后由收集器聚合以最终产生汇总的百分位值(percentile，即平常说的 P99/P50/Pxx)。例如，Upsteam 响应时间。

在 Envoy 的内部实现中，Counters 和 Gauges 被分批并定期刷新以提高性能，Histograms 在接收时写入。

从指标的产出地点来划分，可以分为：

- cluster manager: 面向 upstream 的 L3/L4/L7 层指标。
- http connection manager(HCM)： 面向 upstream & downstream 的 L7 层指标。
- listeners: 面向 downstream 的 L3/L4 层指标。
- server：全局。

Envoy 通过 admin 端口放开了 prometheus 格式的监控指标，可以通过 admin 接口快速查看当前已支持的指标：<http://127.0.0.1:9901/stats/prometheus>，其中 9901 是 admin 端口。
