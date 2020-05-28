---
title: Artillery教程
comments: true
toc: true
toc_number: false
copyright: true
mathjax: false
katex: false
hide: false
date: 2020-05-28 00:39:24
tags:
	- code
	- javascript
	- tutorial
categories: javascript
description: 用Artillery做load测试，分分钟点爆API！快速上手
cover: https://raw.githubusercontent.com/8128/PicGo/master/20200528004132.png
top_img:
keywords:
---

最近写了个SQS服务，是说在我们系统调用docusign第三方服务的时候，假如docusign的burst limit被点爆了，我们就得把request放SQS那边去。这个手动测试就比较麻烦了，毕竟要把docusign先点爆了，才能让我发送的request送到SQS去。

如何点爆一个API呢，实际上[Artillery](https://artillery.io/docs/getting-started/)就是一个很方便的工具。安装过程我直接放官方的：

Once Node.js is installed, install Artillery with:

```
npm install -g artillery
```

To check that the installation succeeded, run:

```
artillery -V
```

你要是很简单的只是一个快速测试，没有详细需求，可以直接这么用：

```shell
artillery quick --count 10 -n 20 https://artillery.io/
```

然后详细使用方法，我先放几行代码然后在代码后方用注释解释一下：

```yml
config:
  target: "https://demo.docusign.net" #你要点爆的target
  phases: #表示不同阶段，注意，每个阶段都会走完一整个scenario流程
    - duration: 60 #维持时间为60s
      arrivalRate: 400 #并发数400，同时有400个用户在瞎jb点这个target
  processor: "./getEnvelope.js" #你的自定义javascript文件，这里面可以放函数，比方说我里面有genJWT这个函数，然后我module.exports了它，我接下来就可以用export名调用这个函数
scenarios:
  - name: "get envelope for account accountId:xxxxxx"
    flow:
      - function: "genJWT" #我调用的函数名，同时这是这个workflow的第一步。注意，整个workflow都会被一起重新运行，你添加phase并不会说是第一个phase跑第一个flow。
      - post: # 一个post请求
          url: "https://account-d.docusign.com/oauth/token"
          afterResponse: "logResponse" # 也是自定义函数，接收返回的信息
          json:
            grant_type: "urn:ietf:params:oauth:grant-type:jwt-bearer"
            assertion: "{{ jwt }}" # 这是从我的prosessor中返回的一个值
          capture:
            json: "$.access_token"
            as: "access_token"
      - get:
          url: "/restapi/v2.1/accounts/xxxxxxxx" #这是一个相对路径，会被加在target后面。这也是flow的第三部
          afterResponse: "logResponse"
          headers:
            authorization: "Bearer {{ access_token }}"
```

然后其中提到的processor，我是这么写的：

```js
async function genJWT(context, events, done) {
  const jwt = 'xxxxxxxx';
  context.vars['jwt'] = jwt; //在这里我相当于定义了环境变量，Artillery就能调用这个jwt了
  return done();
}

async function logResponse(requestParams, response, context, ee, next) {
  console.log(response.headers); //在这里我打印了我的headers，我用这个headers来看burst limit还剩多少
}

module.exports = {
  genJWT,
  logResponse,
}
```

应该解释得还算清楚吧

