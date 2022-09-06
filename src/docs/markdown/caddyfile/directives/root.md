---
title: root (Caddyfile指令)
---

# root

设置网站的根路径，由各种匹配器和访问文件系统的指令使用。如果不设置，默认的网站根目录是当前工作目录。

具体来说，这个指令设置了`{http.vars.root}`占位符。它与同一区块中的其他`root'指令是相互排斥的，所以用相交的匹配器定义多个根是安全的：它们不会级联和相互覆盖。

该指令不会自动启用静态文件的服务，所以它通常与[`file_server`指令](/docs/caddyfile/directives/file_server)或[`php_fastcgi`指令](/docs/caddyfile/directives/php_fastcgi)一起使用。


## 语法

```caddy-d
root [<matcher>] <path>
```

- **&lt;path&gt;** 是用于网站根目录的路径。

注意`<path>`参数如果以`/`开头，可能会被解析器混淆为[匹配器标记]（/docs/caddyfile/matchers#syntax）。为了消除混淆，可以指定一个通配符匹配器标记（`*`）。见下面的例子。

## 示例

为所有请求设置网站根目录为`/home/user/public_html`。

(注意，这里需要一个[通配符匹配器](/docs/caddyfile/matchers#wildcard-matchers)，因为第一个参数与[路径匹配器](/docs/caddyfile/matchers#path-matchers)不明确。)

```caddy-d
root * /home/user/public_html
```

为所有请求设置网站根目录为`public_html`（相对于当前工作目录）。

(这里不需要匹配器标记，因为我们的网站根目录是一个相对路径，所以它不是以正斜杠开始的，因此不会产生歧义。)

```caddy-d
root public_html
```

只为`/foo/*`中的请求改变网站根。

```caddy-d
root /foo/* /home/user/public_html/foo
```

root "指令通常与[`file_server`](/docs/caddyfile/directives/file_server)配对，为静态文件提供服务，或者与[`php_fastcgi`](/docs/caddyfile/directives/php_fastcgi)配对，为PHP网站提供服务。

```caddy-d
root * /home/user/public_html
file_server
```
