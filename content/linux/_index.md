---
title: 'Linux'
description: 'Linux 下工作常用命令、工具、软件总结。'
date: 2021-01-29T23:54:37+08:00
draft: false
---

Linux 下工作常用命令、工具、软件总结。

## Ubuntu

Ubuntu 查询指定软件有多少可用版本:

```console
apt-cache madison <<package name>>
```

安装软件时指定版本号：

```console
 $ apt-get install <<package name>>=<<version>>
 # apt-get install kubeadm=1.20.7-00
```

查看 Ubuntu 版本代号：

```console
$ sb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description: Ubuntu 20.04.2 LTS
Release: 20.04
Codename: focal
```
