---
title: Caddyfile教程
---

# Caddyfile教程

本教程将教你[HTTP Caddyfile](/docs/caddyfile)的基础知识，以便你可以快速轻松地生成美观、功能强大的站点配置。

**目标：**
- 🔲 第一个站点
- 🔲 静态文件服务器
- 🔲 模板
- 🔲 压缩
- 🔲 多个站点
- 🔲 匹配器
- 🔲 环境变量
- 🔲 注释

**先决条件：**
- 基本的终端/命令行技能
- 基本的文本编辑器技能
- PATH变量包含`caddy`

---

新建一个名为`Caddyfile`（无扩展名）的文本文件。

首先应该输入的是你的网站[地址](/docs/caddyfile/concepts#addresses)。

```caddy
localhost
```

<aside class="tip">
    如果HTTP和HTTPS端口（分别为80和443）是你操作系统上的特权端口，你将需要以提升的特权运行或使用更高的端口。要使用更高的端口，只需将其更改为类似<code>localhost:2015</code>的地址，并使用Caddyfile的<a href="/docs/caddyfile/options">http_port</a>选项更改HTTP端口。	
</aside>

然后按回车键并输入你想要它执行的操作。对于本教程，使你的Caddyfile如下所示：

```caddy
localhost

respond "Hello, world!"
```

保存并运行Caddy（因为这是一个培训教程，我们将使用该`--watch`标志，以便自动应用对Caddyfile的更改）：

<pre><code class="cmd bash">caddy run --watch</code></pre>

<aside class="tip">
    如果你遇到权限错误，请尝试在你的地址中使用更高的端口（如<code>localhost:2015</code>）并<a href="/docs/caddyfile/options">更改HTTP端口</a>，或者提升权限后再次运行。
</aside>

第一次，系统会要求你输入密码。这样Caddy就可以通过HTTPS为你的网站提供服务。

<aside class="tip">
只要主机或IP是站点地址的一部分，Caddy默认通过HTTPS为所有站点提供服务。<a href="/docs/automatic-https">自动HTTPS</a>可以通过显式地为地址添加前缀<code>http://</code>予以禁用。
</aside>

<aside class="complete">第一个站点</aside>

在你的浏览器中使用HTTPS浏览[https://localhost](https://localhost)，检查你的网站服务器是否正常运行。

<aside class="tip">
    如果你第一次遇到证书错误，你可能需要重新启动浏览器。
</aside>

这并不是特别令人兴奋，所以让我们将静态响应更改为启用目录列表的[文件服务器](/docs/caddyfile/directives/file_server) ：

```caddy
localhost

file_server browse
```

保存你的 Caddyfile，然后刷新你的浏览器页面。如果当前目录中有索引文件，你应该会看到文件列表或HTML页面。

<aside class="complete">静态文件服务器</aside>

## 添加功能

利用文件服务器还可以做一些更有意思事情：基于模板页面展示。创建一个新文件并粘贴如下内容：

```html
<!DOCTYPE html>
<html>
	<head>
		<title>Caddy tutorial</title>
	</head>
	<body>
		Page loaded at: {{`{{`}}now | date "Mon Jan 2 15:04:05 MST 2006"{{`}}`}}
	</body>
</html>
```

将其在当前目录保存为`caddy.html`，然后在浏览器中访问：[https://localhost/caddy.html](https://localhost/caddy.html)

输出内容如下：

```
Page loaded at: {{`{{`}}now | date "Mon Jan 2 15:04:05 MST 2006"{{`}}`}}
```

等一下，我们不是应该能看到今天的日期吗？为什么它不起作用呢？这是因为：服务器尚未配置模板！小事一桩，只需在Caddyfile中添加一行即可：

```caddy
localhost

templates
file_server browse
```

保存它，然后刷新浏览器页面，你将会看到：

```
Page loaded at: {{now | date "Mon Jan 2 15:04:05 MST 2006"}}
```

使用Caddy的[模板模块](/docs/modules/http.handlers.templates)，你可以对静态文件做很多有用的事情，例如包含其他HTML文件、制作子请求、设置响应头、处理数据结构等等！

<aside class="complete">模板</aside>

使用快速且现代的压缩算法压缩响应是一种很好的做法。使用 [`encode`](/docs/caddyfile/directives/encode)可以启用Gzip和Zstandard支持：

```caddy
localhost

encode zstd gzip
templates
file_server browse
```

<aside class="tip">浏览器还不支持Zstandard编码。希望很快能支持了！</aside>

<aside class="complete">压缩</aside>

这是启动和运行半高级、可应用于生产的站点的基本过程！

当你准备好开启[自动HTTPS](/docs/automatic-https)时，只需将你的网站地址（`localhost`在我们的教程中）替换为你的域名即可。有关更多信息，请参阅我们的[HTTPS快速入门指南](/docs/quick-starts/https)。

## 多个站点

使用我们当前的Caddyfile，我们只能有一个站点定义！只有第一行可以是站点的地址，然后文件的所有其余部分都必须是该站点的指令。

但是制作起来很容易，所以我们可以添加更多网站！

到目前为止，我们的Caddyfile内容如下：

```caddy
localhost

encode zstd gzip
templates
file_server browse
```

相当于这个：

```caddy
localhost {
	encode zstd gzip
	templates
	file_server browse
}
```

这样就能添加两个甚至更多的站点了。

通过将我们的站点块包裹在花括号`{ }`中，我们可以在同一个Caddyfile中定义多个不同的站点。

例如：

```caddy
:8080 {
	respond "I am 8080"
}

:8081 {
	respond "I am 8081"
}
```

当用花括号包裹站点块时，只有[地址](/docs/caddyfile/concepts#addresses)出现在花括号外，[指令](/docs/caddyfile/directives)都出现在花括号内。

对于共享相同配置的多个站点，你可以添加更多地址，例如：

```caddy
:8080, :8081 {
	...
}
```

然后，你可以根据需要定义任意数量的不同站点，只要每个地址都是唯一的。

<aside class="complete">多个站点</aside>


## 匹配器

我们可能只想将某些指令应用于某些请求。例如，假设我们想要同时拥有一个文件服务器和一个反向代理，但我们显然不能在每个请求上都这样做！文件服务器将写入静态文件，或者反向代理将请求代理到后端。

这个配置不会像我们想要的那样工作：

```caddy
localhost

file_server
reverse_proxy 127.0.0.1:9005
```

在实践中，我们可能只想对 API 请求使用反向代理，即基本路径为`/api/`。通过添加[匹配器标记](/docs/caddyfile/matchers#syntax)很容易做到这一点：

```caddy
localhost

file_server
reverse_proxy /api/* 127.0.0.1:9005
```

这里，现在反向代理只会处理所有以`/api/`开始的请求。

我们刚刚添加的`/api/*`标记就被称为**匹配器标记**，你可以说它是一个匹配器标记，因为它以正斜杠开头，并且出现在指令之后（但你始终可以在[指令文档](/docs/caddyfile/directives)中查找它以确定）。

匹配器真的很强大。你可以命名匹配器并使用它们`@name`来匹配不仅仅是请求路径！在继续之前花点时间了解更多关于[匹配器](/docs/caddyfile/matchers)的信息！

<aside class="complete">匹配器</aside>

## 环境变量

Caddyfile适配器允许在解析Caddyfile之前替换[环境变量](/docs/caddyfile/concepts#environment-variables)。

首先，设置一个环境变量（在运行Caddy的同一shell中）：

<pre><code class="cmd bash">export SITE_ADDRESS=localhost:9055</code></pre>

然后你可以在Caddyfile中这样使用它：

```caddy
{$SITE_ADDRESS}

file_server
```

在解析Caddyfile之前，它将被扩展为：

```caddy
localhost:9055

file_server
```

你可以在Caddyfile中的任何位置使用环境变量，使用数量也没有限制。

<aside class="complete">环境变量</aside>


## 注释

最后一件你会发现最有帮助的事情：如果你想在你的Caddyfile中添加注释或注释任何东西，你可以将`#`放在行首：

```caddy
# this starts a comment
```

<aside class="complete">注释</aside>

## 进一步阅读

- [常见模式](/docs/caddyfile/patterns)
- [Caddyfile概念](/docs/caddyfile/concepts)
- [指令](/docs/caddyfile/directives)