---
title: push (Caddyfile指令)
---

# push

配置服务器使用HTTP/2服务器预先向客户端推送资源。

通过指定响应的Link头，可以为服务器推送资源提供链接。这条指令将自动推送由上游Link头描述的资源，格式如下。

- `<resource>; as=script`
- `<resource>; as=script,<resource>; as=style`
- `<resource>; nopush`
- `<resource>;<resource2>;...`

其中`<resource>`以正斜线`/`开头（也就是说，是一个具有相同主机的URI路径）。只有同主机的资源菜可以被推送。如果有属于外部资源的路径，或者标识了`nopush`的属性，它将不会被推送。

默认情况下，推送请求将包括一些被认为可以从原始请求中安全复制的头信息。

- Accept-Encoding
- Accept-Language
- Accept
- Cache-Control
- User-Agent

如果没有这些头信息，许多请求都会失败；这些header头是不需要手动配置的。

推送请求在内部被虚拟化，所以它们是非常轻量的。


## 语法

```caddy-d
push [<matcher>] [<resource>] {
	[GET|HEAD] <resource>
	headers {
		[+]<field> [<value|regexp> [<replacement>]]
		-<field>
	}
}
```

- **&lt;resource&gt;** 是要推送的目标URI路径。如果在区块内使用，可以选择在前面加上方法（GET或POST；默认是GET）。
- **&lt;headers&gt;** 使用与[`header`指令](/docs/caddyfile/directives/header)相同的语法来操作推送请求的头信息。有些头信息是默认带入的，不需要明确配置（见上文）。


## 示例

在响应中推送任何由`Link`头信息描述的资源。

```caddy-d
push
```

和上面一样，但也向所有请求推送`/resources/style.css`。

```caddy-d
push * /resources/style.css
```

只有在客户端请求`/foo.html`时才推送`/foo.jpg`。

```caddy-d
push /foo.html /foo.jpg
```
