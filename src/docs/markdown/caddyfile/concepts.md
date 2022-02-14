---
title: Caddyfile概念
---

# Caddyfile概念

本文档将帮助你详细了解HTTP Caddyfile。

1. [结构](#structure)
2. [地址](#addresses)
3. [匹配器](#matchers)
4. [占位符](#placeholders)
5. [片段](#snippets)
6. [注释](#comments)
7. [环境变量](#environment-variables)
8. [全局选项](#global-options)


## 结构

下面的图片可以直观地描述Caddyfile的结构：

![Caddyfile的结构](/resources/images/caddyfile-visual.png)

关键点：

- 可选的**全局选项块**可以放在文件的头部
- 否则, Caddyfile的首行**总是**要提供服务的网站地址。
- 所有指令和匹配器都**必须**放在站点块中。跨站点块没有全局范围或继承。
- 如果**只有一个站点块**，则其花括号`{ }`是可选的。

一个Caddyfile至少包含一个或多个站点块，这些块总是以站点的一个或多个[地址](#addresses)开始。出现在地址之前的任何指令都会使扰乱解析器。

### 块

打开和关闭一个**块**是用花括号完成的：

```
... {
	...
}
```

- 打开的花括号`{`必须位于行尾。
- 大括号`}`必须独占一行。

当只有一个站点块时，花括号（和缩进）是可选的。这是为了方便快速定义单个站点，例如：

```caddy
localhost

reverse_proxy /api/* localhost:9001
file_server
```

相当于：

```caddy
localhost {
	reverse_proxy /api/* localhost:9001
	file_server
}
```

当你只有一个站点块时；这是一个偏好问题。

要使用相同的Caddyfile配置多个站点，你必须在每个站点周围使用花括号来分隔它们的配置：

```caddy
example1.com {
	root * /www/example.com
	file_server
}

example2.com {
	reverse_proxy localhost:9000
}
```

如果一个请求匹配多个站点块，则选择具有最具体匹配地址的站点块。请求不会级联到其他站点块。

### 指令

[**指令**](/docs/caddyfile/directives)是自定义网站服务方式的关键字。例如，完整的文件服务器配置可能如下所示：

```caddy
localhost

file_server
```

或反向代理：

```caddy
localhost

reverse_proxy localhost:9000
```

在这些示例中，[`file_server`](/docs/caddyfile/directives/file_server)和[`reverse_proxy`](/docs/caddyfile/directives/reverse_proxy)是指令。指令是站点块中一行的第一个单词。

在第二个示例中，`localhost:9000`是一个**参数**，因为它出现在指令之后的同一行。

请注意，当调整 Caddyfile 时，指令会根据特定的默认[指令顺序](/docs/caddyfile/directives#directive-order)进行排序。

**子指令**可以出现在指令块中：

```caddy
localhost

reverse_proxy localhost:9000 localhost:9001 {
	lb_policy first
}
```

这里，`lb_policy`是[`reverse_proxy`](/docs/caddyfile/directives/reverse_proxy)的子指令（它用于设置后端之间使用的负载平衡策略）。


### 标记和引号

Caddyfile在被解析之前被词法解析成标记。空格在Caddyfile中很重要，因为标记就是由它进行分隔。

通常，指令需要一定数量的参数。如果单个参数有一个带有空格的值，它会被作为两个单独的标记进行词法分析：

```caddy-d
directive abc def
```

这可能会出现问题并返回错误或意外行为。

如果`abc def`应该是单个参数的值，则需要使用引号：

```caddy-d
directive "abc def"
```

如果你也需要在带引号的标记中使用引号，则可以转义引号：

```caddy-d
directive "\"abc def\""
```

在带引号的标记内，所有其他字符都按字面意思处理，包括空格、制表符和换行符。

你还可以使用反引号`来引用标记：

```caddy-d
directive `"foo bar"`
```

当标记包含引号文字时，反引号字符串很方便，例如JSON文本。


## 地址

地址总是出现在站点块的顶部，并且通常出现在Caddyfile中的第一行。

这些是有效地址的示例：

- `localhost`
- `example.com`
- `:443`
- `http://example.com`
- `localhost:8080`
- `127.0.0.1`
- `[::1]:2015`
- `example.com/foo/*`
- `*.example.com`
- `http://`

<aside class="tip">
    如果你的站点地址包含主机名或 IP 地址，则会启用<a href="/docs/automatic-https">自动HTTPS</a>。然而，这种行为纯粹是隐含的，因此它永远不会覆盖任何显式配置。例如，如果站点的地址是<code>http://example.com</code>，则不会激活自动HTTPS，因为该方案是明确的<code>http://</code>。	
</aside>

从地址中，Caddy可以潜在地推断出你站点的方案、主机、端口和路径。

如果你指定主机名，则只会处理具有匹配Host标头的请求。换句话说，如果站点地址是`localhost`，那么Caddy将不会匹配到`127.0.0.1`的请求。

可以使用通配符(`*`)，但仅代表主机名的一个标签。例如，`*.example.com`匹配`foo.example.com`，但不匹配`foo.bar.example.com`，`*`匹配`localhost`，但不匹配`example.com`。要捕获所有主机，请省略地址的主机部分。

如果多个站点共享相同的定义，你可以将所有站点一起列出：

```caddy
localhost:8080, example.com, www.example.com
```

或者

```caddy
localhost:8080,
example.com,
www.example.com
```

请注意逗号如何表示地址的延续。

地址必须是唯一的；你不能多次指定同一个地址。



## 匹配器

默认情况下，注入HTTP处理程序的指令适用于所有请求（除非另有说明）。

请求匹配器可用于按给定标准对请求进行分类。这个概念源于[底层的JSON结构](/docs/json/apps/http/servers/routes/match/)，知道如何在Caddyfile中使用它们很重要。使用匹配器，你可以准确指定某个指令适用于哪些请求。

对于支持匹配器的指令，指令后的第一个参数是**匹配器标记**。这里有些例子：

```caddy-d
root *           /var/www  # matcher token: *
root /index.html /var/www  # matcher token: /index.html
root @post       /var/www  # matcher token: @post
```

完全省略匹配器标记，则可以匹配所有的请求；例如，如果下一个参数看起来不像路径匹配器，则不需要给出`*`。

**[阅读有关请求匹配器的页面](/docs/caddyfile/matchers)，了解更多信息。**




## 占位符

你可以在Caddyfile中使用任何[Caddy占位符](/docs/conventions#placeholders)，但为方便起见，你还可以使用一些等效的速记符：

| 简写                                    | 替换                                                 |
|---------------------------------------|----------------------------------------------------|
| `{dir}`                               | `{http.request.uri.path.dir}`                      |
| `{file}`                              | `{http.request.uri.path.file}`                     |
| `{header.*}`                          | `{http.request.header.*}`                          |
| `{host}`                              | `{http.request.host}`                              |
| `{labels.*}`                          | `{http.request.host.labels.*}`                     |
| `{hostport}`                          | `{http.request.hostport}`                          |
| `{port}`                              | `{http.request.port}`                              |
| `{method}`                            | `{http.request.method}`                            |
| `{path}`                              | `{http.request.uri.path}`                          |
| `{path.*}`                            | `{http.request.uri.path.*}`                        |
| `{query}`                             | `{http.request.uri.query}`                         |
| `{query.*}`                           | `{http.request.uri.query.*}`                       |
| `{re.*.*}`                            | `{http.regexp.*.*}`                                |
| `{remote}`                            | `{http.request.remote}`                            |
| `{remote_host}`                       | `{http.request.remote.host}`                       |
| `{remote_port}`                       | `{http.request.remote.port}`                       |
| `{scheme}`                            | `{http.request.scheme}`                            |
| `{uri}`                               | `{http.request.uri}`                               |
| `{tls_cipher}`                        | `{http.request.tls.cipher_suite}`                  |
| `{tls_version}`                       | `{http.request.tls.version}`                       |
| `{tls_client_fingerprint}`            | `{http.request.tls.client.fingerprint}`            |
| `{tls_client_issuer}`                 | `{http.request.tls.client.issuer}`                 |
| `{tls_client_serial}`                 | `{http.request.tls.client.serial}`                 |
| `{tls_client_subject}`                | `{http.request.tls.client.subject}`                |
| `{tls_client_certificate_pem}`        | `{http.request.tls.client.certificate_pem}`        |
| `{tls_client_certificate_der_base64}` | `{http.request.tls.client.certificate_der_base64}` |
| `{upstream_hostport}`                 | `{http.reverse_proxy.upstream.hostport}`           |



## 片段

你可以定义称为片段的特殊块，方法是给它们一个用括号括起来的名称：

```caddy
(redirect) {
	@http {
		protocol http
	}
	redir @http https://{host}{uri}
}
```

然后你可以在任何你需要的地方重复使用它：

```caddy-d
import redirect
```

[`import`](/docs/caddyfile/directives/import) 指令还可用于在其位置包含其他文件。作为一种特殊情况，它几乎可以出现在Caddyfile中的任何位置。

你可以将参数传递给导入的配置并像这样使用它们：

```caddy
(snippet) {
  respond "Yahaha! You found {args.0}!"
}

a.example.com {
	import snippet "Example A"
}

b.example.com {
	import snippet "Example B"
}
```


## 注释

注释从行首的`#`开始并一直持续到行尾：

```caddy-d
# Comments can start a line
directive  # or go at the end
```

哈希字符`#`不能出现在标记的中间（即它必须以空格开头或出现在行首）。这允许在URI或其他值中使用它而不需要引号。

## 环境变量

如果你的配置依赖于环境变量，你可以在Caddyfile中使用它们：

```caddy
{$SITE_ADDRESS}
```
这种形式的环境变量在解析开始之前被替换，因此它们可以扩展为空值、部分标记、完整标记，甚至是多个标记和行。

当未找到环境变量时，可以指定默认值，方法是使用`:`变量名和默认值之间的分隔符：

```caddy
{$DOMAIN:localhost}
```

如果你想将环境变量的替换推迟到运行时，你可以使用[标准`{env.*}`占位符](/docs/conventions#placeholders)。


## 全局选项

Caddyfile可以选择以没有键的特殊块开始，称为[全局选项块](/docs/caddyfile/options)：

```caddy
{
	...
}
```

如果存在，它必须是配置中的第一个块。

它用于设置全局适用的选项，或不适用于任何特定站点。在里面，只能设置全局选项；你不能在其中使用常规站点指令。

点击[了解](/docs/caddyfile/options)有关全局选项块的更多信息。