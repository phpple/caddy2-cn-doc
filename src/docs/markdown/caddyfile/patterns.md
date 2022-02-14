---
title: 常见Caddyfile模式
---

# 常见Caddyfile模式

此页面演示了一些用于常见用例的完整和最小的Caddyfile配置。这些可能是你自己的Caddyfile文档的有用起点。

这些不是即插即用的解决方案；你将不得不自定义你的域名、端口/套接字、目录路径等。它们旨在说明一些最常见的配置模式。

#### Menu

- [静态文件服务器](#静态文件服务器)
- [反向代理](#反向代理)
- [PHP](#PHP)
- [重定向到`www.`子域名](#重定向到`www.`子域名)
- [尾随斜杠](#尾随斜杠)
- [通配符](#通配符)
- [单页面应用(SPAs)](#单页面应用(SPAs))


## 静态文件服务器

```caddy
example.com

root * /var/www
file_server
```

像往常一样，第一行是站点地址。该[`root`](/docs/caddyfile/directives/root)指令指定站点根目录的路径（`*`匹配所有请求的方法，以便与[路径匹配器](/docs/caddyfile/matchers#path-matchers)消除歧义）；如果站点不是当前工作目录，则更改站点的路径。最后，我们启用静态文件服务器。


## 反向代理

代理所有的请求：

```caddy
example.com

reverse_proxy localhost:5000
```

只代理以`/api/`开头的请求，并为其他所有内容提供静态文件：

```caddy
example.com

root * /var/www
reverse_proxy /api/* localhost:5000
file_server
```


## PHP

在运行PHP FastCGI服务的情况下，类似这样的内容适用于大多数现代PHP应用程序：

```caddy
example.com

root * /var/www
php_fastcgi /blog/* localhost:9000
file_server
```

请对应地调整站点根目录和路径匹配器；此示例假定PHP仅位于`/blog/`子目录中——所有其他请求将作为静态文件提供。

该[`php_fastcgi`指令](/docs/caddyfile/directives/php_fastcgi)实际上只是[几个配置](/docs/caddyfile/directives/php_fastcgi#expanded-form)的快捷方式。


## 重定向到`www.`子域名

T使用HTTP重定向，给域名**添加**`www.`子域。

```caddy
example.com {
	redir https://www.example.com{uri}
}

www.example.com {
}
```

要**删除**它：

```caddy
www.example.com {
	redir https://example.com{uri}
}

example.com {
}
```


## 尾随斜杠

你通常不需要自己进行配置；该[`file_server`指令](/docs/caddyfile/directives/file_server)将通过 HTTP 重定向自动添加或删除请求中的尾部斜杠，具体取决于请求的资源是目录还是文件。

但是，如果需要，你仍然可以在配置中强制使用斜杠。有两种方法可以做到：内部或外部。

### 内部执法

这使用[`rewrite`](/docs/caddyfile/directives/rewrite)指令。Caddy将在内部重写URI以添加或删除尾部斜杠：

```caddy
example.com

rewrite /add     /add/
rewrite /remove/ /remove
```

使用重写，带有和不带有斜杠的请求将是相同的。


### 外部执行

这使用[`redir`](/docs/caddyfile/directives/redir)指令。Caddy 将要求浏览器更改 URI 以添加或删除尾部斜杠：

```caddy
example.com

redir /add     /add/
redir /remove/ /remove
```

使用重定向，客户端将不得不重新发出请求，为资源强制执行一个可接受的URI。

## 通配符证书

如果你需要使用相同的通配符证书为多个子域提供服务，处理它们的最佳方法是使用如下所示的Caddyfile，利用[`handle`](/docs/caddyfile/directives/handle)指令和[`host`][`host`](/docs/caddyfile/matchers#host)匹配器：

```caddy
*.example.com {
	tls {
		dns <provider_name> [<params...>]
	}

	@foo host foo.example.com
	handle @foo {
		respond "Foo!"
	}

	@bar host bar.example.com
	handle @bar {
		respond "Bar!"
	}

	# Fallback for otherwise unhandled domains
	handle {
		abort
	}
}
```

请注意，你必须启用[ACME DNS 质询](/docs/automatic-https#dns-challenge)才能让 Caddy 自动管理通配符证书。


## 单页面应用(SPAs)

当一个网页进行自己的路由时，服务器可能会收到大量的请求，这些请求并不存在于服务器端，但只要提供单一的索引文件，这些请求就可以在客户端呈现。这样架构的Web应用程序被称为SPA，或单页应用程序。

其主要思想是让服务器“尝试文件”来查看请求的文件在服务器端是否存在，如果不存在，则返回到一个索引文件，客户端在其中进行路由(通常使用客户端JavaScript):`try_files {path} /index.html`

最基本的SPA配置通常是这样的：

```caddy
example.com

root * /path/to/site
try_files {path} /index.html
file_server
```

如果你的SPA与API或其他仅在服务器端使用的端点相结合，你会想要使用`handle`块来专门处理它们：

```caddy
example.com

handle /api/* {
	reverse_proxy backend:8000
}

handle {
	root * /path/to/site
	try_files {path} /index.html
	file_server
}
```
