---
title: 'nanano'
date: 2020-12-14T23:54:37+08:00
draft: true
---

网站定时更新镜像版本。

增加定时任务，定时执行脚本。

```bash
vi /etc/crontab

# add new line
# 0 5 * * * root /usr/bin/bash /xxx/nightly-update.sh

# reload crontab
systemctl reload crond.service
# show crontab list
crontab -l
```

nightly-update.sh 内容如下。

```bash
#!/bin/bash
TAG=nightly

docker pull nxest/idaas-book:${TAG}
docker pull nxest/bits-pieces:${TAG}
docker pull nxest/l10178-github-io:${TAG}

kubectl delete pod -n bits-pieces -l app.kubernetes.io/name=bits-pieces
kubectl delete pod -n idaas-book -l app.kubernetes.io/name=idaas-book
kubectl delete pod -n l10178-github-io -l app.kubernetes.io/name=l10178-github-io

# clean images with none tag
sleep 120
docker images | grep -w none | awk '{print $3}' | xargs docker rmi

```
