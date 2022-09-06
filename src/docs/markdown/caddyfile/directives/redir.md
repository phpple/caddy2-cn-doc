---
title: redir (Caddyfile指令)
---

# redir

向客户发出一个HTTP重定向。

这条指令意味着一个匹配的请求将被拒绝。它在处理程序链中被安排在很早的位置（在[`rewrite`](/docs/caddyfile/directives/rewrite)之前）。

## 语法

```caddy-d
redir [<matcher>] <to> [<code>]
```

- **&lt;to&gt;** 是目标位置。成为响应的Location的header头。
- **&lt;code&gt;** 是重定向要使用的HTTP状态码。可以是。
	- 3xx范围内的一个正整数，或401
	- `temporary` 用于临时重定向（默认为302）
	- `permanent` 用于永久重定向（301）
	- `html` 使用一个HTML文档来执行重定向（对重定向浏览器很有用，但对重定向API客户端没有用）
	- 一个带有状态代码值的占位符



## 示例

将所有请求重定向到`https://example.com`。

```caddy-d
redir https://example.com
```

和上面一样，但保留URI不变：

```caddy-d
redir https://example.com{uri}
```

相同，但是是永久重定向：

```caddy-d
redir https://example.com{uri} permanent
```

将旧的`/about-us`页面重定向到新的`/about`页面。

```caddy-d
redir /about-us /about
```
