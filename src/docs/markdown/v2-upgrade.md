---
title: 升级到Caddy 2
---

升级指南
=============

为了改进Caddy 1，Caddy 2从头开始编码，是一个全新的代码库。Caddy 2不向后兼容Caddy 1。但也不用太担心，大多数基本设置其实并没有太大不同。本指南将帮助你尽可能平滑地过渡。

本指南不会深入研究可用的新功能——虽然它们真的很酷，顺便说一下，你应该学习它们——这里的目标是让你快速启动并运行 Caddy 2。

### 菜单

- [要点](#high-order-bits)
- [步骤](#steps)
- [HTTPS和端口](#https-and-ports)
- [命令行](#command-line)
- [Caddyfile](#caddyfile)
	- [主要变化](#primary-changes)
	- [basicauth](#basicauth)
	- [browse](#browse)
	- [errors](#errors)
	- [ext](#ext)
	- [fastcgi](#fastcgi)
	- [gzip](#gzip)
	- [header](#header)
	- [log](#log)
	- [proxy](#proxy)
	- [redir](#redir)
	- [rewrite](#rewrite)
	- [root](#root)
	- [status](#status)
	- [templates](#templates)
	- [tls](#tls)
- [服务文件](#service-files)
- [插接件](#plugins)
- [获取帮助](#getting-help)


## 要点(High-order bits)

- “Caddy 2”仍然只是被称为`caddy`。我们使用“Caddy 2”仅用来描述阐明是哪个版本，使过渡不那么混乱。
- 大多数用户只需要替换他们的`caddy`二进制文件和更新的`Caddyfile`配置（在测试它是否有效之后）。
- 最好不要从`Caddy 1`继承任何假设进入`Caddy 2`。
- 你可能无法在 v2 中完美复制你的细分(liche)v1 配置。通常，这是有充分理由的。
- 不再用命令行进行服务器配置。
- 配置不再需要环境变量。
- 为Caddy 2提供配置的主要方法是通过其[API](/docs/api)，但也可以使用[`caddy`命令](/docs/command-line)。
- 你应该知道Caddy 2的原生配置语言是[JSON](/docs/json/)，而Caddyfile只是另一个为你转换为 JSON 的[配置适配器](/docs/config-adapters)。并非所有可能的配置都可以由Caddyfile表示，一些极端情况下的自定义、或者高级使用场景下可能需要JSON。
- Caddyfile基本相同，但功能更强大；指令已更改。



## 步骤(steps)

1. 通过我们的[入门](/docs/getting-started)教程熟悉Caddy 2 。
2. 如果你还没有，请执行第1步。说真的——我们不得不强调至少要知道如何使用Caddy 2的重要性。（它更有趣！）
3. 使用以下指南转换你的`caddy`命令。
4. 使用以下指南转换你的`Caddyfile`.
5. 在本地或暂存中测试你的新配置。
6. 测试，测试，再测试
7. 部署并玩得开心！


## HTTPS和端口(and ports)

Caddy的默认端口不再是`:2015`。Caddy 2 的默认端口是`:443`，而如果不知道主机名或者IP，端口为`:80`。你始终可以在配置中自定义端口。

如果主机名或者IP是已知的，则Caddy 2的默认协议[_总是_ HTTPS](/docs/automatic-https#overview)。这与 Caddy 1 不同，在 Caddy 1 中，默认情况下只有公开域名使用 HTTPS。现在，每个站点都使用 HTTPS（除非你通过明确指定端口`:80`或通过`http://`禁用它)。

IP 地址和localhost域将从[本地受信任的嵌入式 CA](/docs/automatic-https#local-https)颁发证书。所有其他域将使用ZeroSSL或Let's Encrypt。（这都是可配置的。）

证书和 ACME 资源的存储结构发生了变化。Caddy 2 可能会为你的站点获得新证书；但是如果你有很多证书，如果它不适合你，你可以手动迁移它们。有关详细信息，请参阅问题[#2955](https://github.com/caddyserver/caddy/issues/2955)和[#3124](https://github.com/caddyserver/caddy/issues/3124)。



## 命令行(Command line)

`caddy`命令现在是`caddy run`。

所有命令行标志都是不同的。删除它们；所有服务器配置现在都存在于实际配置文档中（通常是 Caddyfile 或 JSON）。你可能会从[JSON结构](/docs/json/)或[Caddyfile全局选项](/docs/caddyfile/options)中找到你需要的内容，对v1中的大多数命令行标志进行替换。

像`caddy -conf ../Caddyfile`这样的命令会变成`caddy run --config ../Caddyfile`。

和以前一样，如果你的Caddyfile在当前文件夹中，Caddy会自动找到并使用它；在这种情况下，你不需要使用`--config`标志。

信号基本相同，只是不再支持USR1和USR2。请改用[`caddy reload`](/docs/command-line#caddy-reload)命令或[API](/docs/api)来加载新配置。

在没有任何配置的情况下运行`caddy`用于运行简单的文件服务器。Caddy 2 中的等价物是[`caddy file-server`](/docs/command-line#caddy-file-server)。

环境变量不再相关，除了`HOME`（并且，可选地，`XDG_*`你设置的任何变量）。`CADDYPATH`被[操作系统约束所替代](/docs/conventions#file-locations)。


## Caddyfile

[v2 Caddyfile](/docs/caddyfile/concepts)与你已经熟悉的非常相似。你需要做的主要事情是更改指令。

⚠️ **请务必阅读新指令！** 特别是如果你的配置更高级，则需要考虑许多细微差别。这些技巧可以让你快速切换，但请阅读每个指令的完整文档，以便了解升级的含义。当然，在将它们投入生产之前，请务必彻底测试你的配置。

### 主要变化

- 如果你提供静态文件服务器，你需要添加一个[`file_server`指令](/docs/caddyfile/directives/file_server)，因为 Caddy 2 默认不假设这个。出于安全原因，默认情况下 Caddy 2 也不嗅探 MIME。如果缺少 Content-Type，你可能需要使用[header](/docs/caddyfile/directives/header)指令自己设置标头。

- 在v1中，你只能按请求路径过滤（或“匹配”）指令。在 v2 中，[请求匹配](/docs/caddyfile/matchers)功能更加强大。任何向 HTTP 处理程序链添加中间件或以任何方式操纵 HTTP 请求/响应的 v2 指令都利用了这个新的匹配功能。[阅读有关v2请求匹配器的更多信息](/docs/caddyfile/matchers)，你需要了解它们才能理解v2的Caddyfile。

- 尽管许多[占位符](/docs/conventions#placeholders)是相同的，但许多已更改，现在有[很多新](/docs/modules/http#docs)占位符，包括[给Caddyfile的简写](/docs/caddyfile/concepts#placeholders)。

- Caddy 2 的日志都是结构化的，默认格式是 JSON。所有日志级别都可以简单地转到要处理的同一日志（但如果需要，你可以自定义）。

- 在 Caddy 1 中通过路径前缀匹配请求的情况下，现在默认情况下 Caddy 2 中的路径匹配是精确的。如果要匹配类似`/foo/`的前缀，则在Caddy 2中需要使用`/foo/*`进行匹配。

我们将在这里列出一些最常见的 v1 指令，并描述如何转换它们以在 v2 Caddyfile 中使用。

⚠️ **仅仅因为此页面有些v1指令没有给出，并不意味着v2不能做到！** 一些 v1 指令不需要，翻译不好，或者在 v2 中以其他方式实现。对于一些高级定制，你可能需要下拉到 JSON 以获得你想要的。浏览[我们的文档](/docs/caddyfile)以找到你需要的内容！


### basicauth

HTTP基本身份验证仍使用该[`basicauth`](/docs/caddyfile/directives/basicauth)指令进行配置。但是，Caddy 2 配置不接受明文密码。你必须对它们进行哈希处理，使用[`caddy hash-password`](/docs/command-line#caddy-hash-password)可以帮助你搞定。

- **v1：**
```
basicauth /secret/ Bob hiccup
```

- **v2：**
```caddy-d
basicauth /secret/* {
	Bob JDJhJDEwJEVCNmdaNEg2Ti5iejRMYkF3MFZhZ3VtV3E1SzBWZEZ5Q3VWc0tzOEJwZE9TaFlZdEVkZDhX
}
```


### browse

现在通过[`file_server`](/docs/caddyfile/directives/file_server)指令启用文件浏览。

- **v1：**
```
browse /subfolder/
```
- **v2：**
```caddy-d
file_server /subfolder/* browse
```


### errors

自定义错误页面可以使用[`handle_errors`](/docs/caddyfile/directives/handle_errors)。


- **v1：**:

```
errors {
	404 404.html
	500 500.html
}
```

- **v2：**:

```
handle_errors {
	rewrite * /{http.error.status_code}.html
	file_server
}
```

### ext

隐含的文件扩展名可以用[`try_files`](/docs/caddyfile/directives/try_files)。

- **v1：** `ext .html`
- **v2：** `try_files {path}.html {path}`


### fastcgi

假设你正在使用 PHP，则v2等效项是[`php_fastcgi`](/docs/caddyfile/directives/php_fastcgi)。

- **v1：**
```
fastcgi / localhost:9005 php
```
- **v2：**
```caddy-d
php_fastcgi localhost:9005
```

请注意，v1的`fastcgi`指令在后台做了很多工作，包括尝试磁盘上的文件、重写请求，甚至重定向。v2的`php_fastcgi`指令也为你做这些事情，但文档提供了它的[扩展形式](/docs/caddyfile/directives/php_fastcgi#expanded-form)，如果你的要求不同，你可以对其进行修改。

v2版本中不需要预设`php`，因为`php_fastcgi`指令默认采用PHP。像`php_fastcgi 127.0.0.1:9000 php`会导致反向代理认为有第二个后端称为`php`，从而导致连接错误。

v2中的子指令不同——你可能不需要任何PHP指令。


### gzip

现在，[`encode`](/docs/caddyfile/directives/encode)指令可以用于所有响应编码，包括多种压缩格式。

- **v1：**
```
gzip
```
- **v2：**
```caddy-d
encode gzip
```

有趣的事实：Caddy 2也支持`zstd` （但还没有浏览器支持）。


### header

[大部分没有改变](/docs/caddyfile/directives/header)，但现在更强大，因为它可以在 v2 中进行子字符串替换。

- **v1：**
```
header / Strict-Transport-Security max-age=31536000;
```
- **v2：**
```caddy-d
header Strict-Transport-Security max-age=31536000;
```


### log

启用访问记录；该[`log`](/docs/caddyfile/directives/log)指令仍然可以在 v2 中使用，但默认情况下，所有日志都是结构化的，编码为 JSON。

启用访问日志的推荐方法很简单：

```caddy-d
log
```

它将结构化日志发送到标准错误。（你也可以发送到文件或网络套接字；请参阅[`log`](/docs/caddyfile/directives/log)指令文档。）

默认情况下，日志将采用[结构化](/docs/logging)JSON 格式。如果由于遗留原因你仍然需要通用日志格式 (CLF) 的日志，你可以使用[`format-encoder`](https://github.com/caddyserver/format-encoder)插件。


### proxy

v2的等效项是[`reverse_proxy`](/docs/caddyfile/directives/reverse_proxy)。

显著的子指令变化分别是：`header_upstream`和`header_downstream`，已经对应地变成了`header_up`和`header_down`；负载平衡相关的子指令则都带上了前缀`lb_`。

另一个显着的区别是v2代理默认通过所有传入的标头（包括`Host`标头）且设置`X-Forwarded-For`标头。换句话说，v1 的“透明”模式基本上是 v2 中的默认模式（但如果你需要 X-Real-IP 等其他标头，则必须自己设置）。你仍然可以使用`header_up`子指令覆盖或自定义`Host`标头。

Websocket 代理在 v2 中“正常工作”；无需像v1那样“启用（enable）”websocket。

得益于请求匹配器的改进，v2不再需要进行[重写hack](#rewrite)，因此`without`自治领已经被移除了。

- **v1：**
```
proxy / localhost:9005
```
- **v2：**
```caddy-d
reverse_proxy localhost:9005
```


### redir

[不变](/docs/caddyfile/directives/redir)，除了一些关于可选状态码参数的细节。大多数配置不需要进行任何更改。

- **v1：** `redir https://example.com{uri}`
- **v2：** `redir https://example.com{uri}`


### rewrite

请求重写（“内部重定向”）的语义略有改变。如果你在v1中使用所谓的“重写hack”进行请求的匹配而不是使用简单路径前缀的方式，那么在v2中这是完全没有必要的。

[新`rewrite`指令](/docs/caddyfile/directives/rewrite)简单而强大，因为它的大部分复杂性都被v2中的[匹配器](/docs/caddyfile/matchers)承接了：

- **v1：**
```
rewrite {
	if {>User-Agent} has mobile
	to /mobile{uri}
}
```
- **v2：**
```caddy-d
@mobile {
	header User-Agent *mobile*
}
rewrite @mobile /mobile{uri}
```

请注意我们如何简单地使用Caddy 2的常用[匹配器指令](/docs/caddyfile/matchers)；它不再是该指令的特例。

首先删除所有的重写hack；将它们变成[命名匹配器](/docs/caddyfile/concepts#named-matchers)。评估每个v1的`rewrite`以查看v2中是否真的需要它。提示：`rewrite`用于添加路径前缀然后删除相同前缀，而带有`without`的`proxy`用来移除相同前缀的写法是一个重写hack，可以去掉了。

你可能会发现新的[`route`](/docs/caddyfile/directives/route)和[`handle`](/docs/caddyfile/directives/handle)指令能更好地控制高级路由逻辑。


### root

[未改变](/docs/caddyfile/directives/root)，但如果你的根路径以`/`开头，则需要添加一个`*`匹配器标记以将其与[路径匹配器](/docs/caddyfile/concepts#path-matchers)区分开来。

- **v1：** `root /var/www`
- **v2：** `root * /var/www`

因为它接受v2的匹配器，这意味着你还可以根据请求更改站点根目录。

如果提供静态文件，请记住添加[`file_server`指令](https://caddyserver.com/docs/caddyfile/directives/file_server)，因为默认情况下 Caddy 2 不假设这一点，而在 v1 中始终启用它。


### status

v2等效的是[`respond`](/docs/caddyfile/directives/respond)，它也可以写一个响应体。

- **v1：**
```
status 404 /secrets/
```
- **v2：**
```caddy-d
respond /secrets/* 404
```


### templates

[`templates`](/docs/caddyfile/directives/templates)指令的整体语法没有改变，但实际的模板动作/功能是不同的并且有很大的改进。例如，模板能够包含文件、渲染 markdown、制作内部子请求、解析前端内容等等！

[查阅文档](/docs/modules/http.handlers.templates)了解新功能的更多细节。

- **v1：** `templates`
- **v2：** `templates`


### tls

[`tls`](/docs/caddyfile/directives/tls)指令的基本原理没有改变，例如指定你自己的证书和密钥：

- **v1：** `tls cert.pem key.pem`
- **v2：** `tls cert.pem key.pem`

但是Caddy的[自动HTTPS逻辑](/docs/automatic-https) _已经_ 已经改变，所以要注意这一点！

密码套件名称也发生了变化。

Caddy 2中的一个常见配置是使用`tls internal`它为非开发主机名`localhost`或 IP 地址提供本地受信任的证书。

大多数网站根本不需要这个指令。


## Service files

我们建议使用[我们的官方 systemd 服务文件](/docs/running#linux-service)之一进行 Caddy 部署。

如果你需要自定义服务文件，请以我们的为基础。出于充分的理由，他们已经仔细调整过！如果需要，请务必自定义你的。


## Plugins

为 v1 编写的插件不会自动与 v2 兼容。v2 甚至不需要许多 v1 插件。另一方面，v2 比 v1 更容易扩展和灵活！

如果你想为 Caddy 2 编写插件，请[学习如何开发Caddy模块](/docs/extending-caddy)。


### 使用插件构建 Caddy 2

Caddy 2 可以在[交互式下载页面](https://caddyserver.com/download)通过插件下载。或者，你可以使用`xcaddy`[自己构建Caddyfile](https://caddyserver.com/docs/build)选择要包含的插件。 `xcaddy`自动执行Caddy的[main.go](https://github.com/caddyserver/caddy/blob/master/cmd/caddy/main.go)文件中的指令。


## 获得帮助(Getting help)

如果你难以让 Caddy 正常工作，请先浏览我们的网站以获取文档。花时间尝试新事物并了解正在发生的事情——v2 在很多方面与 v1 非常不同（但也非常熟悉）！

如果你仍然需要帮助，请加入[我们的社区](https://caddy.community)！你可能会发现帮助他人也是帮助自己的最佳方式。