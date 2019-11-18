---
layout: post
title: "给SHELL设置代理"
video: false
---

Golang在国内无法访问这个应该并不陌生，比如想配置一个`go mobile`的环境，使用`go get golang.org/x/mobile/cmd/gomobile`时，就会因为golang.org被Block的原因失败。如果此时刚好有可以连上golang.org的代理，则事情就变得简单了。

设置环境变量：http_proxy，https_proxy即可。

示例如下：

```
# http://127.0.0.1:8081为真实的代理地址
export set http_proxy=http://127.0.0.1:8081
export set https_proxy=http://127.0.0.1:8081
```

> 使用的时候需要注意，设置https_proxy时，代理的地址也是http的，不然会出现https协商异常。

写于 2018-07-16 16:59:28