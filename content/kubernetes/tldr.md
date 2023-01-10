---
title: 'TL;DR'
date: 2021-01-29T23:54:37+08:00
draft: false
---

复制--粘贴，这就是生活。

---

复制 secret 到另一个 namespace。

```sh
kubectl get secret mys --namespace=na -oyaml | grep -v '^\s*namespace:\s' | kubectl apply --namespace=nb -f -
```

批量删除 pod。

```sh
kubectl get pods --all-namespaces | grep Evicted | awk '{print $2 " --namespace=" $1}' | xargs kubectl delete pod
# Delete by label
kubectl delete pod -n idaas-book -l app.kubernetes.io/name=idaas-book
```

密钥解密。

```sh
 kubectl get secret my-creds -n mysql -o jsonpath="{.data.ADMIN_PASSWORD}" | base64 --decode
```

Docker 保存和导入镜像。

```sh
# save image(s)
docker save image:tag image2:tag | gzip >xxx.tar.gz
# load images
docker load -i xxx.tar.gz
```

合并多个 kube config。

```sh
export KUBECONFIG=~/.kube/config:~/.kube/anotherconfig
kubectl config view --flatten > ~/.kube/config-all

cp ~/.kube/config-all ~/.kube/config
# 顺手把权限改了，避免helm或kubectl客户端warning
chmod 600 ~/.kube/config

```
