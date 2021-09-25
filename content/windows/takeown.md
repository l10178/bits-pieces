---
title: 'Windows提权 + 设置环境变量'
description: 'Windows提权并设置环境变量'
date: 2021-09-25T10:54:37+08:00
draft: false
---

背景：公司 Windows 办公机受域控安全策略限制，部分文件无权修改，另外开发常用的设置系统环境变量也变灰无法设置。此问题解决方式如下。

## 提升文件权限

1. 点击 Windows + X 快捷键 – 选择「命令提示符（管理员）。

2. 在 CDM 窗口中执行如下命令。

   ```cmd
   takeown /f C:\要修复的文件路径
   ```

3. 在拿到文件所有权后，还需要使用如下命令获取文件的完全控制权限。

   ```cmd
   icacls C:\要修复的文件路径 /Grant Administrators:F
   ```

## 命令行设置环境变量

Windows 下命令行设置环境变量，方式为 `setx 变量名 变量值`，变量值带空格等特殊符号的，用引号引起来。

```cmd
# 通过命令行设置 Java Home
setx JAVA_HOME "C:\Program Files\Java\jdk-11.0.2"
# 设置 GO Path
setx GOPATH "D:\workspace\go"
```
