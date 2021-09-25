---
title: 'History入门'
date: 2021-01-29T23:54:37+08:00
draft: false
---

Linux 下 history 命令，用好历史命令提高工作效率。

## 搜索历史命令

快捷键 Ctrl + r

非常建议你使用这个命令, 因为当你曾经输过一个很长的命令之后, 当你再次想输入这个命令的时候, 你就可以按下这个快捷键, 然后键入那条长命令的关键词, 然后就会显示出含有那个关键词的命令, 每次按下这个键都会再往上搜一个。可以找个机器实际体会下，确实很常用。

## 重复上一次的命令

向上的方向键。上下键翻看历史命令，翻到想执行的命令回车。

两个叹号: ！！

还有这个：！-1

快捷键：Ctrl+p 或 Ctrl+n，向上和向下翻看历史命令，和上下键效果一样。

从历史记录中执行某个命令
还是沿袭上一个中的 !-n 模式, 其中 n 是一个编号。如下示例，执行了编号为 4 的命令。

```console
1 service network restart
2 exit
3 id
4 cat /etc/redhat-release
# !4
cat /etc/redhat-release
```

## 执行曾经的命令中特定开头的

假设你的部分历史命令如下:

```console
1721 find . -type f
```

那么, 怎样重复执行 1721 条呢? 除了利用 !-1721 这么麻烦的方法，我们还可以用 !f 这样的姿势。因为开头的 f 是离着最后一条命令最近的, 所以 !f 就执行了它。

## 清空历史记录

```console
history -c
```

## 在历史记录中显示时间

我们可以用 HISTTIMEFORMAT 这个变量来定义显示历史记录时的时间参数:

```console
# export HISTTIMEFORMAT='%F %T '
# history
```

也可以用下面的别名来定义显示历史命令的数量:

```console
alias h1='history 10'
```

## 用 HISTCONTROL 来删除重复的历史记录

下面的例子中, 有三个 pwd 命令, 那么在 history 中就会显示三次 pwd , 有点不那么人性化。

```console
# pwd
# pwd
# pwd
# history | tail -4
44 pwd
45 pwd
46 pwd
47 history | tail -4
```

所以我们可以这样修改:

```console
export HISTCONTROL=ignoredups
```

然后就不会出现相邻的重复记录了。

## History 扩展和总结

用这个功能可以选择特定的历史记录, 不论是修改还是立即执行, 都可以完成。

!! 重复上一条命令。

!10 重复历史记录中第 10 条命令。

!-2 重复历史记录中倒数第二条命令。

!string 重复历史记录中最后一条以 string 开头的命令。

!?string 重复历史记录中最后一条包含 string 的命令。
