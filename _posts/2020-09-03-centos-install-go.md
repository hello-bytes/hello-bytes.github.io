---
layout: post
title: "在centos上安装go"
subtitle: ''
tags:
  - 笔记 
  - centos使用
---


本文以安装go 1.15 为例。

## 找一个目录，下载并解压Go

以下示例为在/home/aaron目录下操作

```
wget https://golang.org/dl/go1.15.linux-amd64.tar.gz
tar -xf go1.15.linux-amd64.tar.gz
```

## 拷贝到系统目录，设置环境变量

完成上面的操作后，即可以看到`/home/aaron`目录下有一个`go`目录，将整个目录拷贝到`/usr/local/bin/go`下，完成后`/usr/local/bin`目录结构如下：

```
└── go
    ├── api
    ├── AUTHORS
    ├── bin
    ├── CONTRIBUTING.md
    ├── CONTRIBUTORS
    ├── doc
    ├── favicon.ico
    ├── lib
    ├── LICENSE
    ├── misc
    ├── PATENTS
    ├── pkg
    ├── README.md
    ├── robots.txt
    ├── SECURITY.md
    ├── src
    ├── test
    └── VERSION
```

## 添加环境变量

修改`/etc/profile`文件，设置环境变量，如下所示：

```
export GOROOT=/usr/local/bin/go
export GOPATH=/home/aaron/go
export PATH=$PATH:$GOROOT/bin
```

