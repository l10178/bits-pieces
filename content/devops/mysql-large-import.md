---
title: 'MySQL大文件导入优化'
date: 2021-12-29T23:54:37+08:00
draft: false
---

项目中需要根据SQL文件导入数据，文件大约20G，正常导入约需要2小时，如何加快导入速度。

如果一个SQL文件只有一个表的数据，可以直接使用mysql load data infile 语法，速度比较快。

我们是一个SQL文件包含了很多表，导入过程经过如下设置，20G大约需要40分钟。

```bash
# 进入mysql
mysql -u root -p

# 创建数据库（如果已经有数据库忽略此步骤）
CREATE DATABASE 数据库名;

# 设置参数
set sql_log_bin=OFF;//关闭日志
set autocommit=0;//关闭autocommit自动提交模式 0是关闭  1 是开启（默认）
set global max_allowed_packet = 20 *1024* 1024 * 1024;

# 使用数据库
use 数据库名;

# 开启事务
START TRANSACTION;

# 导入SQL文件并COMMIT（因为导入比较耗时，导入和COMMIT一行命令）
source 文件的路径; COMMIT;

```
