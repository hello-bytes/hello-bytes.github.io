---
layout: post
title: "在centos上安装docker"
subtitle: ''
tags:
  - 笔记
---

依次运行以下命令：

```
cd /etc/yum.repos.d
wget http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

yum -y install docker-ce docker-ce-cli

systemctl restart docker
systemctl enable docker
```

如果没有`wget`命令，得先`yum install wget`安装。

## centos 8上遇到的问题

在centos 8上，并没有得到预期的结果，得到如下错误信息：

```
Error:
 Problem: package docker-ce-3:19.03.12-3.el7.x86_64 requires containerd.io >= 1.2.2-3, but none of the providers can be installed
  - cannot install the best candidate for the job
  - package containerd.io-1.2.10-3.2.el7.x86_64 is filtered out by modular filtering
  - package containerd.io-1.2.13-3.1.el7.x86_64 is filtered out by modular filtering
  - package containerd.io-1.2.13-3.2.el7.x86_64 is filtered out by modular filtering
  - package containerd.io-1.2.2-3.3.el7.x86_64 is filtered out by modular filtering
  - package containerd.io-1.2.2-3.el7.x86_64 is filtered out by modular filtering
  - package containerd.io-1.2.4-3.1.el7.x86_64 is filtered out by modular filtering
  - package containerd.io-1.2.5-3.1.el7.x86_64 is filtered out by modular filtering
  - package containerd.io-1.2.6-3.3.el7.x86_64 is filtered out by modular filtering
```

**错误的原因是：**

centos8的yum库中没有符合最新版docker-ce对应版本的containerd.io，docker-ce-3:19.03.11-3.el7.x86_64需要containerd.io >= 1.2.2-3

因此，需要先安装对应的containerd.io

```
yum install -y https://mirrors.aliyun.com/docker-ce/linux/centos/7/x86_64/edge/Packages/containerd.io-1.2.13-3.2.el7.x86_64.rpm 
```

然后再运行`yum -y install docker-ce docker-ce-cli`即可完成docker的安装。
