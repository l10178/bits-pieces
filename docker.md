# Docker从入门到放弃





## [本地导入/保存/载入镜像](https://www.cnblogs.com/linjiqin/p/8604756.html)
1. 导入本地镜像

有时候我们自己在本地或者其它小伙伴电脑上拷贝了一份镜像，有了这个镜像之后，我们可以把本地的镜像导入，使用docker import 命令。

例如这里下载了一个 alibaba-rocketmq-3.2.6.tar.gz 镜像文件，使用下列命令导入：
```
[root@rock]# cat alibaba-rocketmq-3.2.6.tar.gz | docker import - rocketmq:3.2.6(镜像名自己定义)
[root@rock]# docker images
REPOSITORY                   TAG                 IMAGE ID            CREATED             SIZE
rocketmq                     3.2.6               53925d1cf9f0        23 seconds ago      14MB
hello-world                  latest              725dcfab7d63        4 months ago        1.84kB
```
可以看到导入完成后，docker为我们生成了一个镜像ID，使用docker images也可以看到我们刚刚从本地导入的镜像。

*注意镜像文件必须是tar.gz类型的文件*

2. 保存镜像

我们的镜像做好之后，我们要保存起来，以供备份使用，该怎么做？使用docker save命令，保存镜像到本地。
```
[root@rock]# docker save -o rocketmq.tar rocketmq ##-o：指定保存的镜像的名字；rocketmq.tar：保存到本地的镜像名称；rocketmq：镜像名字，通过"docker images"查看
```

3. 载入镜像

我们有了本地的镜像文件，在需要的时候可以使用docker load将本地保存的镜像再次导入docker中。
```
    docker load --input rocketmq.tar
```
