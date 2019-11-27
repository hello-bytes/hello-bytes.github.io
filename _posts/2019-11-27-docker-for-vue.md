---
layout: post
title: "使用Docker部署VUE框架开发的网站"
video: false
---

vue开的发网站，如果不使用服务端渲染的话，就是一些HTML，以及在用户侧（浏览器）执行的脚本。这种情况下，服务器侧只需要部署一个最简单的HTTP文件服务就好。

按这个部署的要求，使用Docker部署就比较简单了:

- 使用`npm run build`或是其它命令进行本地打包，一般会得到`dist`目录
- 以`nginx`的`docker`为源，复制`dist`目录下的文件到`docker`中即可。

`Dockerfile`示例如下：

```
FROM nginx:latest

WORKDIR /pontus/bin/website-manager-service

COPY ./dist/ /usr/share/nginx/html/
```



