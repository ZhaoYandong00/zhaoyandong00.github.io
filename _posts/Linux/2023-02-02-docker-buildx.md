---
title: docker交叉生成
categories: docker buildx
tags: docker buildx
description: docker交叉生成
---
# 创建配置文件
- `/etc/buildkit/buildkitd.toml`开启http
```
debug = true
# root is where all buildkit state is stored.
root = "/var/lib/buildkit"
# insecure-entitlements allows insecure entitlements, disabled by default.
insecure-entitlements = [ "network.host", "security.insecure" ]
http = true
```
# 创建builder容器
- 创建容器
```
docker buildx create --use --name=mybuilder-cn --driver docker-container --driver-opt image=dockerpracticesig/buildkit:master --config /etc/buildkit/buildkitd.toml
```
- 安装交叉生成器
```
docker run --privileged --rm tonistiigi/binfmt --install all
```
- 启动生成器
```
docker buildx inspect --bootstrap
```