---
title: acme_server (Caddyfile指令)
---

# acme_server

一个嵌入式[ACME协议](https://tools.ietf.org/html/rfc8555)服务器处理程序。这允许一个Caddy实例为任何其他兼容ACME的软件（包括其他Caddy实例）签发证书。

启用后，匹配路径`/acme/*`的请求将由ACME服务器处理。


## 客户端配置

使用ACME服务器的默认值，ACME客户端应该简单地配置为使用`https://localhost/acme/local/directory`作为他们的ACME端点。(`local`是Caddy的默认CA的ID)。


## 语法

```caddy-d
acme_server [<matcher>] {
ca <id
}
```

- **ca**指定用于签署证书的认证机构的ID。默认是`local`，这是Caddy的默认CA，用于本地使用的自签证书，这在开发环境中最常见。对于更广泛地使用，建议指定一个不同的CA以避免混淆。如果指定ID的CA不存在，它将被创建。参见[PKI应用程序全局选项](/docs/caddyfile/options#pki-options)以配置替代的CA。
