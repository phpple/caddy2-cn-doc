---
title: request_body (Caddyfile directive)
---

# request_body

操作或设置对传入请求正文的限制。


## 语法

```caddy-d
request_body [<matcher>] {
  max_size <value>
}
```

- **max_size** 是请求正文允许的最大字节数。它接受[go人性化](https://pkg.go.dev/github.com/dustin/go-humanize#pkg-constants)支持的所有大小值。读取到超过限定的字节数将返回HTTP状态413的错误。


## 示例

将请求正文的大小限制在10兆字节。

```caddy-d
request_body {
  max_size 10MB
}
```
