---
title: handle_errors(Caddyfile指令)
---

# handle_errors

设置错误处理程序。

当正常的HTTP请求处理程序返回一个错误时，正常的处理会停止，错误处理程序会被调用。错误处理程序形成一个路由，就像正常的路由一样，它们可以做任何正常路由可以做的事情。这使得在处理HTTP请求期间的错误时有很大的控制力和灵活性。例如，你可以提供静态错误页面、模板化的错误页面，或者反向代理到另一个后端来处理错误。

请求的上下文将被带入错误路由，所以任何在请求上下文上设置的值，如[site root](root)或[vars](vars)，也将被保留在错误处理程序中。此外，在处理错误时，[新占位符](#placeholders)是可用的。

请注意，某些指令，例如[`reverse_proxy`](reverse_proxy)可能会写一个HTTP状态为错误的响应，将 _不会_ 触发错误路由。

你可以使用[`error`](error)指令，根据自己的路由决策明确地触发一个错误。


## 语法

```caddy-d
handle_errors {
    <directives...>
}
```

- **<directives...>** 是一个HTTP处理程序[指令](/docs/caddyfile/directives)和[匹配器](/docs/caddyfile/matchers)的列表，每行一个。


<h3 id="placeholder">占位符</h3>

以下占位符在处理错误时可用。它们是[Caddyfile速记](/docs/caddyfile/concepts#placeholders)的完整占位符，可以在[HTTP服务器错误路由的JSON文档](/docs/json/apps/http/servers/errors/#routes)中找到。

| 占位符 | 描述 |
|---|---|
| `{err.status_code}` | 推荐的HTTP状态代码 |
| `{err.status_text}` | 与推荐状态代码相关的状态文本 |
| `{err.message}` | 错误信息 |
| `{err.trace}` | 错误的来源 |
| `{err.id}` | 该错误发生的标识符 |


## 示例

基于状态代码的自定义错误页面（例如，404错误的页面称为`404.html`）。注意，当在`handle_errors`中运行时，[`file_server`](file_server)保留了错误的HTTP状态码（假设你事先在网站中设置了一个[site root](/docs/caddyfile/directives/root)）。

```caddy-d
handle_errors {
	rewrite * /{err.status_code}.html
	file_server
}
```

一个单一的错误页面，使用[`templates`](/docs/caddyfile/directives/templates)来定制一个自定义的错误信息。

```caddy-d
handle_errors {
	rewrite * /error.html
	templates
	file_server
}
```

反向代理到一个专业的服务器，该服务器在处理HTTP错误和改善你的心情方面有很高的水平😸：

``caddy-d
```caddy-d
handle_errors {
	rewrite * /{err.status_code}
	reverse_proxy https://http.cat {
		header_up Host {upstream_hostport}
	}
}
```

只需使用[`respond`](/docs/caddyfile/directives/respond)即可返回错误代码和名称

```caddy-d
handle_errors {
    respond "{err.status_code} {err.status_text}"
}
```

要以不同的方式处理特定的错误代码，使用[`expression`](/docs/caddyfile/matchers#expression)匹配器，以及[`handle`](/docs/caddyfile/directives/handle)进行互斥。

```caddy-d
handle_errors {
	@404-410 expression `{err.status_code} in [404, 410]`
	handle @404-410 {
		respond "It's a 404 or 410 error!"
	}

	@5xx expression `{err.status_code} >= 500 && {err.status_code} < 600`
	handle @5xx {
		respond "It's a 5xx error."
	}

	handle {
		respond "It's another error"
	}
}
```
