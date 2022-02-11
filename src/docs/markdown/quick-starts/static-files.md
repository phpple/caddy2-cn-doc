---
title: 静态文件快速入门
---

# 静态文件快速入门

本指南将向你展示如何快速启动并运行可用于生产的静态文件服务器。

**先决条件：**
- 基本的终端/命令行技能
- PATH变量包含`caddy`
- 包含你的网站的目录

---

有两种简单的方法可以让快速文件服务器启动并运行。我们将向你展示两种等效的方法来做同样的事情。

## 命令行

在你的终端中，切换到站点的根目录并运行：

<pre><code class="cmd bash">caddy file-server</code></pre>

如果你收到权限错误，这可能意味着你的操作系统不允许你绑定到低端口——因此请改用高端口：

<pre><code class="cmd bash">caddy file-server --listen :2015</code></pre>

然后在浏览器中打开[localhost](http://localhost)（或[localhost:2015](http://localhost:2015)）访问你的站点！

如果你没有索引文件但想要显示文件列表，请使用以下`--browse`选项：

<pre><code class="cmd bash">caddy file-server --browse</code></pre>

你可以使用另一个文件夹作为站点根目录：

<pre><code class="cmd bash">caddy file-server --root ~/mysite</code></pre>



## Caddyfile

在你站点的根目录中，创建一个名为`Caddyfile`的文件，内容如下：

```caddy
localhost

file_server
```

如果你无权绑定到低端口，请替换`localhost`为`localhost:2015`（或其他一些高端口）。

然后，从同一目录运行：

<pre><code class="cmd bash">caddy run</code></pre>

然后，你可以访问[localhost](https://localhost)（或配置中的任何地址）来查看你的站点！

该 [`file_server`指令](/docs/caddyfile/directives/file_server)有更多选项供你自定义站点。更改Caddyfile后，请确保[重新加载](/docs/command-line#caddy-reload)Caddy（或停止并重新启动）！

如果你没有索引文件但想要显示文件列表，请使用以下`browse`参数：

```caddy
localhost

file_server browse
```

你还可以使用另一个文件夹作为站点根目录：

```caddy
localhost

root * /home/me/mysite
file_server
```
