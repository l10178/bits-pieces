---
title: 'Git常用配置'
date: 2023-03-09T23:54:37+08:00
draft: false
---

## Git 多用户配置

Git 给不同目录配置不同的 config，比如区分个人开发账号和公司开发账号。

为账户 B 准备一个单独的配置文件，比如： `~/.gitconfig-b`，内容根据需要定义。

```ini
[user]
	name = userb-name
	email = userb-email@test.com
```

修改 `~/.gitconfig` 文件，增加以下配置，引用上面创建的配置文件，注意用绝对路径，并且路径以 / 结尾。

```ini
[includeIf "gitdir:/project/path-b/"]
    path = /Users/xxxx/.gitconfig-b
```

保存后，在 /project/path-b/ 下新的仓库都会以 .gitconfig-b 中的用户名和邮箱提交了。
