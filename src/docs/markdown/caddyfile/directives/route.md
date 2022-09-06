---
title: route (Caddyfile指令)
---

# route

将一组指令按字面意思作为一个单元进行评估。

路由块中包含的指令不会在内部被重新排序。只有HTTP处理指令（将处理程序或中间件添加到链中的指令）可以在路由块中使用。

这个指令是一个特例，它的子指令也是常规指令。


## 语法

```caddy-d
route [<matcher>] {
	<directives...>
}
```

- **<directives...>** 是一个指令或指令块的列表，每行一个，就像在路由块外面一样；只是这些指令不会被重新排序。只有HTTP处理指令可以被使用。



## 实用性

`路由`指令在某些高级用例或边缘用例中很有帮助，可以对HTTP处理程序链的部分进行绝对控制。

因为HTTP中间件的评估顺序很重要，Caddyfile通常会在解析后重新排列指令，以使Caddyfile更容易使用；你不必担心你输入东西的顺序。

虽然内置的顺序与大多数网站兼容，但有时你需要对顺序进行手动控制，可以是整个网站，也可以只是其中的一部分。这就是 "路由 "指令的用处。

为了说明这一点，考虑两个终止处理程序的情况。`redir`和`file_server`。这两个处理程序都向客户端写入响应，并且不调用链中的下一个处理程序，所以对于某个请求，只有其中一个会被执行。哪个先执行？通常情况下，`redir`在`file_server`之前执行，因为通常你只想在特定情况下发出重定向，在一般情况下提供文件。

然而，在有些情况下，第二个指令（`redir`）比第二个指令（`file_server`）有更具体的匹配器。换句话说，你想在一般情况下重定向，而只服务于一个特定的文件。

所以你可以尝试这样的Caddyfile（但这不会像预期的那样工作！）。

```caddy
example.com

file_server /specific.html
redir https://anothersite.com{uri}
```

问题是，在内部，`redir`排在`file_server`之前，但在这种情况下，`redir`的匹配器是`file_server`的匹配器的超集（`*`是`/specific.html`的超集）。

幸运的是，解决方案很简单：只要把这两个指令包在一个`路由`块中。

```caddy
example.com

route {
	file_server /specific.html
	redir https://anothersite.com{uri}
}
```

<aside class="tip">
另一种方法是使两个匹配器相互排斥，但如果有一个或两个以上的条件，这很快会变得复杂。使用`route`指令，两个处理程序的互斥性是隐含的，因为它们都是终端处理程序。
</aside>

而现在`file_server`将在`redir`之前被链入，因为这个顺序是按字面意思来的。

## 类似的指令

还有其他一些指令可以包装HTTP处理程序指令，但每个指令都有其用途，取决于你想表达的行为。

- [`handle`](handle)像`route`那样包装其他指令，但有两个区别。
  - 1）handle块是相互排斥的
  - 2）通常handle中的指令是[重新排序的](/docs/caddyfile/directives#directive-order)。
- [`handle_path`](handle_path)的作用与`handle`相同，但它在运行其处理程序之前从请求中剥离了一个前缀。
- [`handle_errors`](handle_errors)和`handle`一样，但只有当Caddy在处理请求时遇到错误才会调用。

## 示例

在代理所有API请求到后端之前，从请求路径中去除`/api`前缀。

```caddy-d
route /api/* {
	uri strip_prefix /api
	reverse_proxy localhost:9000
}
```
