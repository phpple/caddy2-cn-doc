---
title: templates (Caddyfile指令)
---

# templates

将响应体作为[template](/docs/modules/http.handlers.templates)文件执行。模板提供了制作简单动态页面的功能基元。功能包括HTTP子请求、HTML文件包含、Markdown渲染、JSON解析、基本数据结构、随机性、时间等。


## 语法

```caddy-d
templates [<matcher>] {
	mime    <types...>
	between <open_delim> <close_delim>
	root    <path>
}
```

- **mime** 是模板中间件将采取行动的MIME类型；任何没有限定Content-Type的响应将不会被评估为模板。默认：`text/html text/plain`。
- **between** 是模板操作的开端和结束定界符。默认值：`{{printf "{{ }}"}}`。如果它们妨碍了你的文档的其他部分，你可以改变它们。
- 当使用访问文件系统的函数时，**root** 是网站根。


## 示例

在所有请求中启用模板。

```caddy-d
templates
```

关于一个网站使用模板提供markdown服务的完整例子，请看一下[这个网站](https://github.com/caddyserver/website)的源代码！特别是，看看[`Caddyfile`](https://github.com/caddyserver/website/blob/master/Caddyfile)和[`src/docs/index.html`](https://github.com/caddyserver/website/blob/master/src/docs/index.html)。
