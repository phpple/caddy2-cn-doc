---
title: uri (Caddyfile指令)
---

处理一个请求的URI。它可以剥离路径前缀/后缀或替换整个URI的子串。

这个指令与[`rewrite`](重写)不同，`uri`是有区别地改变URI，而不是像`rewrite`那样把它重设为完全不同的东西。虽然`rewrite`被特别视为内部重定向，但`uri`只是另一个中间件。


## 语法

支持多种不同的操作：

```caddy-d
uri [<matcher>] strip_prefix <target>
uri [<matcher>] strip_suffix <target>
uri [<matcher>] replace      <target> <replacement> [<limit>]
uri [<matcher>] path_regexp  <target> <replacement>
```

- 第一个（非匹配器）参数指定操作。
	- **strip_prefix**将前缀从路径中剥离。
	- **strip_suffix**将后缀从路径中剥离。
	- **replace**在整个URI中进行子串替换。
	- **path_regexp**在URI的路径部分执行正则表达式替换。
- **&lt;target&gt;**是前缀、后缀、或搜索字符串/正则表达式。如果是前缀，前面的正斜杠可以省略，因为路径总是以正斜杠开始。
- **&lt;replacement&gt;**是替换字符串（只对`replace`和`path_regexp`有效）。支持使用带有`$name`或`${name}`语法的捕获组，或带有数字的索引，如`$1`。详见[Go文档](https://golang.org/pkg/regexp/#Regexp.Expand)。
- **&lt;limit&gt;**是对最大替换次数的可选限制（只对`replace`有效）。

URI的突变发生在URI的规范化或未转义的形式上。然而，转义序列可以在前缀或后缀模式中使用，只匹配请求路径中那些位置的文字转义。例如，`uri strip_prefix /a/b`将把`/a/b/c`和`/a%2Fb/c`改写为`/c`；`uri strip_prefix /a%2Fb`将把`/a%2Fb/c`改写为`/c`，但不会匹配`/a/b/c`。

URI路径在修改前会被清理掉目录遍历的点。此外，除非`<target>`也包含多个斜线，否则多个斜线（如`//`）会被合并。

## 类似指令

其他一些指令也可以对请求URI进行操作。

- [`rewrite`](重写)将整个路径和查询改为新的值，而不是部分地改变值。
- [`handle_path`](handle_path)与[`handle`](handle)的操作相同，但它在运行其处理程序之前从请求中剥离了一个前缀。在许多情况下，可以代替`uri strip_prefix`，以消除额外的一行配置。


## 示例

将`/api`从所有请求路径的开头剥离：

```caddy-d
uri strip_prefix /api
```

从所有请求路径的结尾去除`.php`：

```caddy-d
uri strip_suffix .php
```

在任何请求URI中用"/v1/docs/"替换"/docs/"：

```caddy-d
uri replace /docs/ /v1/docs/
```

将请求路径中所有重复的斜线（但不是请求查询）折叠成一个斜线：

```caddy-d
uri path_regexp /{2,} /
```
