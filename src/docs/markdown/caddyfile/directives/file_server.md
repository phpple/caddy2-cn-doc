---
title: file_server(Caddyfile指令)
---

# file_server

一个静态文件服务器，支持真实和虚拟文件系统。它通过将请求的URI路径附加到[站点的根路径](/docs/caddyfile/directives/root)来形成文件路径。

默认情况下，它执行规范的URI；这意味着HTTP重定向将被用于对不以尾部斜线结尾的目录的请求（添加它）或对有尾部斜线的文件的请求（删除它）。然而，如果内部重写修改了路径的最后一个元素（文件名），则不会发出重定向。

最常见的是，`file_server`指令与[`root`](/docs/caddyfile/directives/root)指令配对，为整个网站设置文件根。一个站点的根并不带有沙盒保证：文件服务器确实可以防止目录遍历，但是根中的符号链接仍然可以允许根之外的访问。

## 语法

```caddy-d
file_server [<matcher>] [browse] {
	fs            <backend...>
	root          <path>
	hide          <files...>
	index         <filenames...>
	browse        [<template_file>]
	precompressed <formats...>
	status        <status>
	disable_canonical_uris
	pass_thru
}
```

- **browse** 对没有索引文件的目录的请求，启用文件列表。
- **fs**指定了一个备用的（也许是虚拟的）文件系统来使用。`caddy.fs`命名空间中的任何Caddy模块都可以在这里使用，只要它支持[`Stat()`调用](https://pkg.go.dev/io/fs#StatFS)。任何根`path/prefix`仍将适用于替代的文件系统模块。默认情况下，使用本地磁盘。
- **root**设置网站根目录。它类似于[`root`](/docs/caddyfile/directives/root)指令，只是它只适用于这个文件服务器实例，并优先于任何可能已经定义的其他站点根目录。默认：`{http.vars.root}`或当前工作目录。注意：这个子指令只改变这个指令的根。其他指令（如[`try_files`](/docs/caddyfile/directives/try_files)或[`templates`](/docs/caddyfile/directives/templates)）要指定根目录，请使用[`root`](/docs/caddyfile/directives/root)指令而不是这个子指令。
- **hide**是一个要隐藏的文件或文件夹的列表；如果要求，文件服务器将假装它们不存在。该指令接受占位符和glob模式。注意，这些是 _文件系统_ 路径，不是请求路径。换句话说，相对路径使用当前工作目录作为基础，而不是网站根目录；所有的路径在比较之前都会被转换为绝对形式（如果可能的话）。指定一个没有路径分隔符的文件名或模式，将隐藏所有具有匹配名称的文件，无论其位置如何；否则，将试图进行路径前缀匹配，然后再进行全局匹配。由于这是一个Caddyfile配置，默认情况下，活动的配置文件将被添加。
- **index**是一个寻找索引文件的文件名列表。默认：`index.html index.txt`。
- **<template_file>**是一个可选的自定义模板文件，用于目录列表。默认为可以找到的模板[在源代码这里！[外部链接](/resources/images/external-link.svg)](https://github.com/caddyserver/caddy/blob/master/modules/caddyhttp/fileserver/browse.html)。浏览模板也可以使用[标准模板模块](/docs/modules/http.handlers.templates#docs)中的动作。
- **precompressed**是用于搜索预压缩挎包文件的编码格式列表。参数是一个有序的编码格式列表，用于搜索预压缩的[sidecar文件]（https://en.wikipedia.org/wiki/Sidecar_file）。支持的格式有`gzip`（`.gz`），`zstd`（`.zst`）和`br`（`.br`）。

  所有的文件查找将首先寻找未压缩文件的存在。一旦找到，Caddy将寻找具有每种启用格式的文件扩展名的sidecar文件。如果找到一个预压缩的sidecar文件，Caddy会用预压缩的文件来回应，并适当地设置`Content-Encoding`响应头。否则，Caddy将以正常的未压缩文件进行响应。如果[`encode`指令](/docs/caddyfile/directives/encode)被启用，那么如果没有预压缩，它可能会对响应进行即时压缩。
- **status**是一个可选的状态代码覆盖，在编写响应时使用。在用自定义错误页面响应请求时特别有用。可以是一个3位数的状态代码，例如：`404`。支持占位符。默认情况下，写入的状态代码通常是`200`，或`206`，用于部分内容。
- **disable_canonical_uris** 如果请求路径是一个目录，则禁用重定向的默认行为，即添加尾部斜线，如果请求路径是一个文件，则删除尾部斜线。请注意，默认情况下，如果请求路径的最后一个元素（文件名）经历了内部重写，那么规范化将不会发生，以避免用隐性行为破坏显式重写。
- **pass_thru** 启用pass-thru模式，如果没有找到请求的文件，则继续到路由中的下一个HTTP处理程序，而不是返回`404`。实际上，这可能只在[`route`](/docs/caddyfile/directives/route)块中有用，因为`file_server`指令实际上是[最后排序](/docs/caddyfile/directives#directive-order)，否则的话。

## 示例

当前目录外的静态文件服务器。

```caddy-d
file_server
```

启用了文件列表:

```caddy-d
文件_服务器浏览
```

只服务于`/static`文件夹中的静态文件:

```caddy-d
file_server /static/*
```

`file_server`指令通常与[`root`指令](/docs/caddyfile/directives/root)配对，以设置提供文件的根路径。

```caddy-d
root * /home/user/public_html
文件服务器
```

隐藏所有`.git`文件夹及其内容。

```caddy-d
file_server {
	隐藏.git
}
```

如果客户端支持（`Accept-Encoding'头），则检查请求的文件是否存在预压缩的文件。因此，如果`/path/to/file`被请求，它将依次检查`/path/to/file.zst`、`/path/to/file.br`和`/path/to/file.gz`，并提供第一个具有相应内容编码的可用文件。

```caddy-d
file_server {
	预压缩 zstd br gzip
}
```
