---
title: request_header (Caddyfile指令)
---

# request_header

处理请求中的HTTP标头字段。它可以设置、添加和删除标头值，或使用正则表达式执行替换。

## Syntax

```caddy-d
request_header [<matcher>] [[+|-]<field> [<value>|<find>] [<replace>]]
```

- **&lt;field&gt;** 是标头字段的名称。默认情况下，将覆盖任何现有的同名字段。
  - 带前缀`+`的字段将会被添加而不是被替换；
  - 带前缀`-`的字段则会被删除。
- **&lt;value&gt;** 是标头字段值，被添加或设置。
- **&lt;find&gt;** 是要搜索的子字符串或正则表达式。
- **&lt;replace&gt;** 是替换值；如果执行搜索和替换，则需要。


## 示例

从请求中删除Referer标头：

```caddy-d
request_header -Referer
```

删除请求中所有包含下划线的header头：

```caddy-d
request_header -*_*
```