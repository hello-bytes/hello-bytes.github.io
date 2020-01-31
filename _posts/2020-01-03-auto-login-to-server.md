---
layout: post
title: "如何配置SSH协议的自动登录"
subtitle: '无密码通过ssh协议自动登录到linux服务器，安全又便捷'
tags:
  - ssh
  - 运维
---

## 1. 客户端创建`rsa key`

```
ssh-keygen -t rsa
```

在运行过程中，全有若干提示选项，比如是否设置密码，全部默认值即可（即遇到让你选择的地方，就按回车键）。

> 在运行此命令前，建议先检查下是否已经有了，对于我的mac电脑来说，检查的方法也很简单，先`cd ~/.ssh`，然后`ls`查看，有没有名为**id_rsa**与**id_rsa.pub**的文件，有则可以忽略这一步。

## 2. 在服务器打开**RSA**连接许可

在服务器上，检查文件`/etc/ssh/sshd_config`中的以下3行是否有被注释（以#开头表示被注释）：

```
PubkeyAuthentication yes
AuthorizedKeysFile yes
AuthorizedKeysFile ~/.ssh/authorized_keys
```

如果被注释了打开就好（即删除前面的`#`）。

## 3. 添加本地公钥到服务器上

如第二步我们注意的三个配置项所示，授权自动登录的信息都放在文件`~/.ssh/authorized_keys`下，所以我们要把客户端的公钥放到服务端的`authorized_keys`文件里。即：

把第一步生成的`id_rsa.pub`文件内容，拷贝到服务器上第二步指定的文件（`~/.ssh/authorized_keys`）里



