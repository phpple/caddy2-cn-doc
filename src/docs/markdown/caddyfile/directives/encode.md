---
title: encode (Caddyfile指令)
---

# encode

使用指定的编码对响应进行编码。其典型用途就是压缩。

## 语法

```caddy-d
encode [<matcher>] <formats...> {
# 编码格式
gzip [<级别>]
zstd

	minimum_length <length

	# 响应匹配器的单行语法
	match [header <field> [<value>]] | [status <code...>]
	# 或者使用多条件的响应匹配器块
	match {
		status <code...>
		header <field> [<value>]
	}
}
```

- **&lt;formats...&gt;** 是要启用的编码格式列表。如果启用了多种编码，则根据请求的Accept-Encoding头来选择编码；如果客户端没有强烈的偏好（q-factor），则使用第一个支持的编码。
- **gzip** <span id="gzip"/> 启用Gzip压缩，可选择指定级别。
- **zstd** <span id="zstd"/>启用Zstandard压缩。
- **minimum_length** <span id="minimum_length"/> 响应编码的最小字节数（默认：512）。
- **match** <span id="match"/>是一个[响应匹配器]（#response-matcher）。只对匹配的响应进行编码。默认的匹配器如下所示：

  ```caddy-d
  match {
      header Content-Type text/*
      header Content-Type application/json*
      header Content-Type application/javascript*
      header Content-Type application/xhtml+xml*
      header Content-Type application/atom+xml*
      header Content-Type application/rss+xml*
      header Content-Type image/svg+xml*
  }
  ```

## 响应匹配器

**响应匹配器**可用于按特定标准过滤（或分类）响应。

### status

```caddy-d
status <code...>
```

按HTTP状态代码。

- **&lt;code...&gt;**是一个HTTP状态代码的列表。特殊情况是`2xx`, `3xx`, ...分别与200-299, 300-399, ...范围内的所有状态码匹配。

### header

参见[header](/docs/caddyfile/matchers#header)请求匹配器的支持语法。

## 示例

启用Gzip压缩。

```caddy-d
encode gzip
```

启用Zstandard和Gzip压缩(Zstandard隐含地优先，因为它是第一个)。

```caddy-d
encode zstd gzip
```
