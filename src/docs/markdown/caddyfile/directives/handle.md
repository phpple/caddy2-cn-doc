---
title: handle(Caddyfile指令)
---

# handle

评估一组指令，这些指令与同级嵌套的其他`handle`块相互排斥。

`handle`指令有点类似于nginx配置中的`location`指令：第一个匹配的`handle`块将被评估。如果需要，处理程序块可以嵌套。只有HTTP处理指令可以在处理块中使用。

## 语法

```caddy-d
handle [<matcher>] {
	<directives...>
}
```

- **<directives...>** 是一个HTTP处理指令或指令块的列表，每行一个，就像在handle块之外使用一样。

## 使用场景

- 如果你喜欢用像nginx位置块那样的基于继承的方式来设计HTTP处理程序逻辑，你可能更喜欢使用`handle`块，而不是为你的指令定义相互排斥的匹配器。
- 如果继承性是你的HTTP处理程序配置所需要的特征，那么`handle`指令可能很适合你。

## 类似的指令

还有其他一些指令可以包装HTTP处理程序指令，但每个指令都有其用途，这取决于你想要表达的行为。

- [`handle_path`](handle_path)的作用与`handle`相同, 但它在运行其处理程序之前从请求中剥离了一个前缀。
- [`handle_errors`](handle_errors)和`handle`一样，但只有当Caddy在处理请求时遇到错误才会调用。
- [`route`](route)像`handle`一样包装其他指令，但有两个区别：
  - 1）路由块之间不相互排斥，
  - 2）路由中的指令不[重新排序](/docs/caddyfile/directives#directive-order)，在需要时给你更多控制。

## 示例

由静态文件服务器处理`/foo/`中的请求，并将所有其他请求发送到反向代理。

```caddy-d
handle /foo/* {
    file_server
}
handle {
    reverse_proxy 127.0.0.1:8080
}
```

你可以在同一个网站中混合使用`handle`和[`handle_path`](handle_path)指令，它们仍然是相互排斥的。

```caddy-d
handle_path /foo/* {
    # 路径有"/foo"前缀的将被去除
}

handle /bar/* {
    # 路径仍然保留了"/bar"
}
```
