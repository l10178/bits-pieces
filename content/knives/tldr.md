---
title: 'TL;DR'
date: 2021-11-20T22:54:00+08:00
draft: false
---

Too Long; Didn’t Read.

tldr 根据二八原则，简化了烦琐的 man 指令帮助文档，仅列出常用的该指令的使用方法，让人一看就懂，大多数情况下，给出几个指令的使用 demo 可能正是我们想要的。

举个例子看下实际运行效果，如下。

```console
➜  ~ tldr docker

  Manage Docker containers and images.
  Some subcommands such as `docker run` have their own usage documentation.
  More information: <https://docs.docker.com/engine/reference/commandline/cli/>.

  List currently running docker containers:

      docker ps

  List all docker containers (running and stopped):

      docker ps -a

  Start a container from an image, with a custom name:

      docker run --name container_name image

  Start or stop an existing container:

      docker start|stop container_name

  Pull an image from a docker registry:

      docker pull image

  Open a shell inside a running container:

      docker exec -it container_name sh

  Remove a stopped container:

      docker rm container_name

  Fetch and follow the logs of a container:

      docker logs -f container_name
```

[tldr](https://github.com/tldr-pages/tldr) 命令行有多种实现，比如官方推荐的有 npm 和 python。

```bash
npm install -g tldr
pip3 install tldr
```

个人更喜欢 Rust 版本的实现 [tealdeer](https://github.com/dbrgn/tealdeer)，支持各系统包管理器和二进制安装，比如 homebrew。

```bash
brew install tealdeer
```
