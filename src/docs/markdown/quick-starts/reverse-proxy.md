---
title: 反向代理快速入门
---

# 反向代理快速入门

本指南将向你展示如何快速启动并运行可用于生产的反向代理。

**先决条件：**
- 基本的终端/命令行技能
- PATH变量支持`caddy`
- 要代理到的正在运行的后端进程

---

有两种简单的方法可以快速启动并运行反向代理。下面将分别介绍这两种等效的方法如何完成同样的事情。

本教程假设你有一个运行在`127.0.0.1:9000`的HTTP后端服务。

## 命令行

在你的终端中，运行以下命令：

<pre><code class="cmd bash">caddy reverse-proxy --to 127.0.0.1:9000</code></pre>

如果你没有绑定到低端口的权限，则可以从较高的端口进行代理：

<pre><code class="cmd bash">caddy reverse-proxy --from :2016 --to 127.0.0.1:9000</code></pre>

然后访问[localhost](https://localhost) （或者你通过`--from`指定的任意地址）发出请求，检查它是否能正常工作！


## Caddyfile

在当前工作目录中，创建一个名为`Caddyfile`的文件，内容如下:

```caddy
localhost

reverse_proxy 127.0.0.1:9000
```

然后，从同一目录执行如下命令：

<pre><code class="cmd bash">caddy run</code></pre>

然后，你可以访问[https://localhost](https://localhost)，检查它是否能正常工作！

要更改代理地址也很简单：

```caddy
:2016

reverse_proxy 127.0.0.1:9000
```

更改Caddyfile后，请确保重新[重载](/docs/command-line#caddy-reload)Caddy（或停止并重新启动它）。

现在你可以通过 [localhost:2016](http://localhost:2016)访问代理了。

你可以使用[`reverse_proxy`指令](/docs/caddyfile/directives/reverse_proxy)完成更多的操作。
