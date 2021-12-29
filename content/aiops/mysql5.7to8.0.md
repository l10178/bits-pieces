---
title: 'MySQL5.7升级至8.0'
date: 2021-12-29T23:54:37+08:00
draft: false
---

MySQL 5.7 升级至 8.0，程序适配以及踩坑记录。

## 代码适配方案

1. mysql-connector-java.jar 升级到 8.0.21 版本。
2. com.mysql.jdbc.Driver 更换为 com.mysql.cj.jdbc.Driver。
3. 链接 url 里指定 useSSL=false。
4. 链接 url 中显式指定时区，增加 serverTimezone=Asia/Shanghai。

## 兼容性问题

1. `com.mysql.jdbc.exceptions.jdbc4` 在 MySQL Connector 8 中不存在。

   将 mysql-connector-java 版本，升级到 8.0 后，如果项目中有使用 mysql 的 Exception 类，编译时会收到以下错误

   ```console
   error: package com.mysql.jdbc.exceptions.jdbc4 does not exist
   [ERROR] import com.mysql.jdbc.exceptions.jdbc4.MySQLSyntaxErrorException;
   [ERROR] import com.mysql.jdbc.exceptions.jdbc4.MySQLIntegrityConstraintViolationException;

   ```

   MySQL 在 8.0 版本重用了现有的 java.sql 异常类，取消了 exceptions.jdbc4 的异常类，整改的异常映射关系如下。
   | 5.7 版本异常类 | 8.0 版本异常类 |
   | ----------- | ----------- |
   |com.mysql.jdbc.exceptions.jdbc4.MySQLSyntaxErrorException | java.sql.SQLSyntaxErrorException|
   |com.mysql.jdbc.exceptions.jdbc4.MySQLIntegrityConstraintViolationException |java.sql.SQLIntegrityConstraintViolationException|
   |com.mysql.jdbc.exceptions.jdbc4.MySQLTransactionRollbackException | java.sql.SQLTransactionRollbackException|

2. 数据库关键字扩充问题。

    在适配 MySQL 8.0 的时候遇到如下报错。

    ```console
    Caused by: java.sql.SQLSyntaxErrorException: You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ', source, i18n_lang' at line 5
    ```

    问题的原因是：MySQL 新增了一批关键字，在使用过程中，如果字段名称和关键字重复，则需要在字段名称上增加单引号方可使用。RANK 为新增关键字。如下所示：

    ```xmll
    <result column="rank" jdbcType="INTEGER" property="rank"/>
    ```

    新增关键字列表如下。

    ```console
    ACTIVE  ,  ADMIN  ,  ARRAY  ,  ATTRIBUTE  ,  BUCKETS  ,  CLONE  ,  COMPONENT  ,  CUME_DIST (R)  ,  DEFINITION  ,  DENSE_RANK (R)  ,  DESCRIPTION  ,  EMPTY (R)  ,  ENFORCED  ,  ENGINE_ATTRIBUTE  ,  EXCEPT (R)  ,  EXCLUDE  ,  FAILED_LOGIN_ATTEMPTS  ,  FIRST_VALUE (R)  ,  FOLLOWING  ,  GEOMCOLLECTION  ,  GET_MASTER_PUBLIC_KEY  ,  GROUPING (R)  ,  GROUPS (R)  ,  HISTOGRAM  ,  HISTORY  ,  INACTIVE  ,  INVISIBLE  ,  JSON_TABLE (R)  ,  JSON_VALUE  ,  LAG (R)  ,  LAST_VALUE (R)  ,  LATERAL (R)  ,  LEAD (R)  ,  LOCKED  ,  MANAGED  ,  MASTER_COMPRESSION_ALGORITH  ,  MSMASTER_PUBLIC_KEY_PATH  ,  MASTER_TLS_CIPHERSUITES  ,  MASTER_ZSTD_COMPRESSION_LEVEL  ,  MEMBER  ,  NESTED  ,  NETWORK_NAMESPACE  ,  NOWAIT  ,  NTH_VALUE (R)  ,  NTILE (R)  ,  NULLS  ,  OF (R)  ,  OFF  ,  OJ  ,  OLD  ,  OPTIONAL  ,  ORDINALITY  ,  ORGANIZATION  ,  OTHERS  ,  OVER (R)  ,  PASSWORD_LOCK_TIME  ,  PATH  ,  PERCENT_RANK (R)  ,  PERSIST  ,  PERSIST_ONLY  ,  PRECEDING  ,  PRIVILEGE_CHECKS_USER  ,  PROCESS  ,  RANDOM  ,  RANK (R)  ,  RECURSIVE (R)  ,  REFERENCE  ,  REQUIRE_ROW_FORMAT  ,  RESOURCE  ,  RESPECT  ,  RESTART  ,  RETAIN  ,  RETURNING  ,  REUSE  ,  ROLE  ,  ROW_NUMBER (R)  ,  SECONDARY  ,  SECONDARY_ENGINE  ,  SECONDARY_ENGINE_ATTRIBUTE  ,  SECONDARY_LOAD  ,  SECONDARY_UNLOAD  ,  SKIP  ,  SRID  ,  STREAM  ,  SYSTEM (R)  ,  THREAD_PRIORITY  ,  TIES  ,  TLS  ,  UNBOUNDED  ,  VCPU  ,  VISIBLE  ,  WINDOW (R)
    ```
