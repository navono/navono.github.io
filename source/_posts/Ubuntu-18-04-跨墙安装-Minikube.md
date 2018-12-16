---
title: Ubuntu 18.04 跨墙安装 minikube
date: 2018-12-16 03:45:40
categories: [笔记]
tags: [docker, k8s, local]
---

# 环境介绍

- Ubuntu 18.04
- Virtual Box: v5.2.10
- ss 和 privoxy
- minikube: v0.30.0
- kubectl: Client: v1.13.0; Server: v1.10.0

首先禁止系统全局的代理.也就是系统设置中 `Network` 中的 `Network Proxy` 设置为 `Off`.如果想要 `Chrome` 上外网,可以使用 `SwitchyOmega` 插件,这样 `Chrome` 上网就不用依赖系统的代理设置了.

# 安装

下载安装 minikube:
> curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
  && sudo install minikube-linux-amd64 /usr/local/bin/minikube

下载安装 kubectl:
> https://github.com/kubernetes/kubernetes/releases
>
> chmod +x kubectl && sudo mv kubectl /usr/local/bin/

# 启动

应为在 minikube 在虚拟机中需要外网下载镜像,所以需要先设置环境变量
> export http_proxy=192.168.99.1:port
>
> export https_proxy=192.168.99.1:port

然后在启动 minikube 时,附加上述的变量:
> minikube start --docker-env HTTP_PROXY=${http_proxy} \
	       --docker-env HTTPS_PROXY=${https_proxy} \
	       --docker-env NO_PROXY=192.168.99.0/24 \
	       --extra-config=apiserver.authorization-mode=RBAC \
	       --kubernetes-version=v1.13.0 \
	       --logtostderr

上述的 `--extra-config=apiserver.authorization-mode=RBAC` 是修改授权规则,应为当前的 minikube(v0.30.0) 的 `dashboard` 无法启动.

启动 `minikube` 用,要使用 `kubectl` 还得设置 `no_proxy` 环境变量:
> export no_proxy=$no_proxy,$(minikube ip)
>
> export NO_PROXY=$no_proxy,$(minikube ip)

# minikube dashboard 无法启动解决方案

等待 minikube 启动后,执行:
> kubectl create clusterrolebinding add-on-cluster-admin --clusterrole=cluster-admin --serviceaccount=kube-system:default

# 测试

> minikube status
>
> kubectl version

使用 `hello-minikube` 测试:
> kubectl run hello-minikube --image=k8s.gcr.io/echoserver:1.10 --port=8080
>
> kubectl expose deployment hello-minikube --type=NodePort
>
> kubectl get pods
>
> curl $(minikube service hello-minikube --url)

删除测试容器:
> kubectl delete services hello-minikube
>
> kubectl delete deployment hello-minikube
