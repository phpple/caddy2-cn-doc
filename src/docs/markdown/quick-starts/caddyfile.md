---
title: Caddyfile 快速入门
---

# Caddyfile 快速入门

创建一个名为Caddyfile（无扩展名）的新文本文件。

最先在Caddyfile输入的内容是你的站点访问地址：

```caddy
localhost
```

<aside class="tip">
    如果 HTTP 和 HTTPS 端口（分别为 80 和 443）是你操作系统上的特权端口，你将需要以提升的特权运行或使用更高的端口。要使用更高的端口，可以将其修改成类似<code>localhost:2015</code>的地址，或者通过Caddyfile的<a href="/docs/caddyfile/options">http_port</a>选项修改HTTP端口。
</aside>

然后按回车键并输入你想要它做的事情，所以它看起来像这样：

```caddy
localhost

respond "Hello, world!"
```

保存并从Caddyfile所在的同一文件夹中运行Caddy：

<pre><code class="cmd bash">caddy start</code></pre>

你可能会被要求输入密码，因为默认情况下，Caddy 通过 HTTPS 为所有站点（甚至本地站点）提供服务。（密码提示应该只在第一次出现！）

<aside class="tip">
    对于本地 HTTPS，Caddy 会自动为你生成证书和唯一的私钥。根证书被添加到系统的信任库中，这就是密码提示的必要性。它允许你通过 HTTPS 在本地进行开发而不会出现证书错误。
</aside>

如果你收到权限错误，可能需要提升权限再次运行。

打开浏览器访问[localhost](http://localhost)或者使用`curl`运行：

<pre><code class="cmd"><span class="bash">curl https://localhost</span>
Hello, world!</code></pre>

你可以通过将它们包裹在花括号`{ }`中来在Caddyfile中定义多个站点。将Caddyfile 更改为：

```caddy
localhost {
	respond "Hello, world!"
}

localhost:2016 {
	respond "Goodbye, world!"
}
```

你可以通过两种方式为Caddy提供更新的配置：直接使用API：

<pre><code class="cmd bash">curl localhost:2019/load \
	-X POST \
	-H "Content-Type: text/caddyfile" \
	--data-binary @Caddyfile
</code></pre>

或使用`reload`命令，它会为你执行相同的API请求：

<pre><code class="cmd bash">caddy reload</code></pre>

在[浏览器](https://localhost:2016)尝试访问新的"goodbye"端点[in your browser]，或者使用`curl`以确保它正常工作：

<pre><code class="cmd"><span class="bash">curl https://localhost:2016</span>
Goodbye, world!</code></pre>

完成 Caddy 后，请务必停止它：

<pre><code class="cmd bash">caddy stop</code></pre>

## 进一步阅读

- [常见模式](/docs/caddyfile/patterns)
- [Caddyfile概念](/docs/caddyfile/concepts)
- [指令](/docs/caddyfile/directives)