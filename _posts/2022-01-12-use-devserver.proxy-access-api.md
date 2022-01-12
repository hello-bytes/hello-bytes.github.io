---
layout: post
title: "使用devServer的proxy功能访问服务端"
subtitle: ''
tags:
  - Vue
  - devServer
  - 跨域
---



> 前后端分离的技术方案下，后端仅需提供相应的API，网页作为客户端直接访问相应接口以完成业务功能。

## 前端本地调试的问题

基于安全考虑，浏览器对跨域请求做了严格的限制，简单说来，就是前端访问后台接口时，要求接口对应API的域名地址和浏览器页面的地址一致才行。

虽然在API侧可以通过在HTTPS请求头部指定特定的属性，允许跨域访问，但基于业务的必要性和安全性考虑，后端接口在上线的时候，一般也不会打开允许跨域的能力。这就为前端的本地开发环境（一般网页地址为http://localhost:8080)，调用开发环境，测试环境，线上环境的API带来了障碍。

为了解决在本地调试页面访问服务端API跨域的问题，可以使用devServer的代理功能。

## 使用devServer的代理功能

当我们运行`npm run dev`的时候，Vue的打包脚本会启动一个本地服务器，此服务器的为`webpack-dev-server`。`webpack-dev-server`本身不属于Vue框架，其是一个标准的HTTP服务器的实现。更多关于`webpack-dev-server`的信息可以参考[https://webpack.docschina.org/configuration/dev-server/](https://webpack.docschina.org/configuration/dev-server/)，在这里我们只需要知道这两点就可以：

- Vue开发框架进行本地调试时，使用的是`webpack-dev-server`来启动一个本地服务器。
- `webpack-dev-server`的配置信息，由配置中的`devServer`这个节来指定。
- `webpack-dev-server`提供了代理的功能。可以将浏览器发过来的请求，转到特定的服务器上。

本文通过配置`devServer`节，来说明如何启用代理以进行联调。对于`Vue 2.0`的配置文件，其路径在`/build/webpack.dev.config.js`目录下。

### devServer节的配置说明

devServer节主要配置服务端的日志级别，端口，本地地址的配置，静态文件（公共文件）的路径等，当然也包括代理功能，示例如下所示：

```
devServer: {
    
    ...

    clientLogLevel: 'warning',
    compress: true,
    host: HOST || config.dev.host,
    port: PORT || config.dev.port,
    publicPath: config.dev.assetsPublicPath,
    proxy: config.dev.proxyTable,
    
    ...

},
```

其中proxy属性的内容，就是代理的配置。

### devServer节的代理配置

proxy的内容是一个以代理地址为key的map，每一个key又对应若干属性。主要的属性值有：


| 字段           | 备注                   | 类型      | 默认值   | 示例                   |
| :----------- | :------------------- | :------ | :---- | :------------------- |
| target       | 代理到的目的地址             | String  | -     | http\://test.env.com |
| changeOrigin | 如果接口跨域，需要进行这个参数配置    | Boolean | true  | true                 |
| secure       | 是否使用HTTPS协议          | Boolean | false | true                 |
| pathRewrite  | URL重写规则，重写URL的Path部分 | Object  | -     |                      |

<pre>
关于URL的Path部分：
         foo://example.com:8042/over/there?name=ferret#nose
         \_/   \______________/\_________/ \_________/ \__/
          |           |            |            |        |
       scheme     authority       path        query   fragment
</pre>

示例如下：

```
proxyTable: {
    '/api_1': {
        changeOrigin: true,
        target: 'http://test1.env.com',
        "secure": false,
        pathRewrite: { '^/api_1': '/api_1' }
    }
}
```

示例解读：

- 当前服务（devServer）收到URL路径为 `/api_1`开头的请求时，按后面的属性进行代理。
- /api_1的请求，被代理到`http://test1.env.com`
- 对于`/api_1`开始的请求，被代理时将去掉`/api_1`，比如`http://localhost:8080/api_1/xxx` => `http://test1.env.com/api_1/xxx`

当然，path也可以被替换,示例如下：

```
proxyTable: {
    '/api_1': {
        changeOrigin: true,
        target: 'http://test1.env.com',
        "secure": false,
        pathRewrite: { '^/api_1': '/test_api' }
    }
}
```

这个配置节和上面的配置节相比，只是修改了`pathRewrite`，命中规则后，会将请求到当前`devServer`的path中的`/api_1`替换为`/test_api`,即`http://localhost:8080/api_1/xxx` => `http://test1.env.com/test_api/xxx`

### 多环境配置

上面一节介绍了代理配置的基本方法。在实际工作中，我们可能会面对各种环境，比如开发环境，测试环境，正式环境。下面举例说明一种配置多环境的方法。

示例如下：

```
const envConfig = {
    dev: {
        proxy: 'https://www.test.baidu.com'
    },
    test: {
        proxy: 'https://test.baidu.com'
    },
    pub: {
        proxy: 'https://pub.baidu.com'
    }
};

const env = envConfig.test;

module.exports = {
    dev: {
        ...

        proxyTable: {
            '/api-prefix': {
                changeOrigin: true,
                target: env.proxy,
                "secure": true,
                pathRewrite: { '^/api-prefix': '/api-prefix' }
        },

        ...
}
```

如上所示，配置了三个环境，分别是`dev`,`test`,`pub`，其域名也各不一样，通过修改变量`env`的指向，即可切换到不同的环境进行开发联调。
