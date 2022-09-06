---
title: respond (Caddyfile指令)
---

# respond

写一个硬编码/静态响应给客户端。


## 语法

```caddy-d
respond [<matcher>] <status>|<body> [<status>] {
	body <text>
	close
}
```

- **&lt;status&gt;** 是要写入的HTTP状态代码。默认为200。
- **&lt;body&gt;** 是要写入的响应体。
- **body** 是提供正文的另一种方式；如果是多行，则很方便。
- **close** 将在写完响应后关闭客户端与服务器的连接。

澄清一下，第一个非匹配器参数可以是一个3位数的状态代码或一个响应体字符串。如果是一个body，下一个参数可以是状态码。

<aside class="tip">
	用错误状态代码响应与在处理程序链中返回错误不同，后者在内部调用错误处理程序。
</aside>


## 示例

给所有的健康检查写一个200状态的空主体。

```caddy-d
respond /health-check 200
```

给所有的请求写一个简单的响应体。

```caddy-d
respond "Hello, world!"
```

写一个错误响应并关闭连接。

```caddy-d
respond /secret/* "Access denied" 403 {
	close
}
```
