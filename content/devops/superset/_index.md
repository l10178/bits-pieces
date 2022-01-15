---
title: 'Superset 对接 ClickHouse'
date: 2021-03-19T23:54:37+08:00
draft: false
---

Apache Superset，开源数据分析与可视化平台。

常用功能：

1. SQL 查询，作为一个 Web 版本的 ClickHouse、MySQL 等等多种数据源的客户端。
2. 支持对查询结果再进行过滤，直接 copy 或下载查询结果，保存查询条件，分享查询条件等。
3. 丰富的图表设计、分析。内置各种 Charts 图表，支持自定义 Charts 和 Dashboard，需要自己根据业务动手制作，这才是他的主业。
4. 支持用户管理，基于 RBAC 模型的控制权限。（理论上可以对接 keycloak，看 issue 还不完善可能需要改 python 代码）。

## 配置文件

全部默认配置文件参考[superset/config.py](https://github.com/apache/superset/blob/master/superset/config.py)。

如果想自定义，创建一个 superset_config.py 文件放到 python path。

## 中文支持

多语言默认是不打开的（因为维护的不好，英文属于正常维护，其他语言属于有空就维护），需要在 superset_config.py 中增加环境变量，支持中文。

```python
# 默认中文
BABEL_DEFAULT_LOCALE='zh'
# 支持很多其他语言，裁掉一部分，只留了这两个
LANGUAGES = {
    "en": {"flag": "us", "name": "English"},
    "zh": {"flag": "cn", "name": "中文"},
}

```
